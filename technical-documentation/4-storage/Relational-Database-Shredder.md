[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Storage**](storage-documentation) > Relational Database Shredder

<a name="overview">

### 1. Overview

Relational Database Shredder (RDB Shredder for short) is a Spark job allowing you to split (shred)
Snowplow enriched events, produced by Spark Enrich into separate enrities. RDB Shredder makes use of
the [scala-common-enrich][sce] project to load enriched events.

You will typically run the RDB Shredder jar as part of an EMR jobflow, started by
[EmrEtlRunner](EmrEtlRunner). It is designed to be used downstream of Spark Enrich and upstream of
[StorageLoader](StorageLoader).

RDB Shredder has two primary tasks:

1. Shred enriched event into `atomic-events` TSV and associated JSONs
2. Make `event_id`s for all events unique

<a name="shredding">

### 2. Shredding

Snowplow [enriched event][EnrichedEvent] is a 131-column TSV file, produced by Spark Enrich. Each
line contains all information about a specific event, including its id, timestamps, custom and
derived contexts and much more.

Shredding is the process of splitting the `EnrichedEvent` TSVs into the following parts:

1. **Atomic events**. a TSV line very similar to `EnrichedEvent` but not contining
   JSON fields (`contexts`, `derived_contexts` and `unstruct_event`). The results
   will be stored in a path similar to
   `shredded/good/run=2016-11-26-21-48-42/atomic-events/part-00000`
   and will be available to load via [StorageLoader](StorageLoader) or directly
   via Redshift [COPY][redshift-copy].
2. **Contexts**. This part consists of the two extracted above JSON fields:
   `contexts` and `derived_contexts`, which are validated (during the enrichment step)
   self-describing JSONs. But, unlike the usual self-describing JSONs consisting of a
   `schema` and a `data` object, these ones consist of a `schema` object
   (like in JSON Schema), the usual `data` object and a `hierarchy` object. This
   `hierarchy` contains data to later join your contexts' SQL tables with the
   `atomic.events` table. The results will be stored in a path which looks like
   `shredded/good/run=2016-11-26-21-48-42/shredded-types/vendor=com.acme/name=mycontext/format=jsonschema/version=1-0-1/part-00000`,
   where the part files like `part-00000` are valid NDJSONs and it will be possible to load them
   via [StorageLoader](StorageLoader) or directly via Redshift [COPY][redshift-copy].
3. **Self-describing (unstructured) events**. Very much similar to the contexts described above
   those are the same JSONs with the `schema`, `data` and `hierarchy` fields. The only difference
   is that there is a one-to-one relation with `atomic.events`, whereas contexts have
   many-to-one relations.

More details on what shredding is can be found on the dedicated [shredding](Shredding) page.

<a name="deduplication">

### 3. Deduplication

Duplicates is a common problem in event pipelines, it has been described and studied
[many][dealing-with-duplicate-event-ids][times][r76-release]. Basically, the
problem is that we cannot guarantee that every event has a unique `UUID` because

1. we have no exactly-once delivery guarantees
2. user-side software can send events more than once
3. we have to rely on flawed [algorithms][issue-2967]

There are four strategies planned regarding incorporating deduplication mechanisms in RDB Shredder:

| Strategy                             | Batch?      | Same event ID? | Same event fingerprint? | Availability                              |
|--------------------------------------|-------------|----------------|-------------------------|-------------------------------------------|
| In-batch natural de-duplication      | In-batch    | Yes            | Yes                     | [R76 Changeable Hawk-Eagle][r76-release]  |
| In-batch synthetic de-duplication    | In-batch    | Yes            | No                      | [R86 Petra][r86-release]                  |
| Cross-batch natural de-duplication   | Cross-batch | Yes            | Yes                     | [R88 Angkor Wat][r88-release]             |
| Cross-batch synthetic de-duplication | Cross-batch | Yes            | No                      | Planned                                   |

We will cover these in turn:

<a name="inbatch-natural-deduplication">

#### 3.1 In-batch natural de-duplication

As of [the R76 Changeable Eagle-Hawk release][r76-release], RDP de-duplicates "natural duplicates"
- i.e. events which share the same event ID (`event_id`) and the same event payload (based by
`event_fingerprint`), meaning that they are semantically identical to each other. For a given ETL
run (batch) of events being processed, RDB Shredder keeps only the first out of each group of
natural duplicates; all others will be discarded.

To enable this functionality you need to have
[the Event Fingerprint Enrichment][fingerprint-enrichment] enabled in order to correctly populate
the `event_fingerprint` property.

<a name="inbatch-synthetic-deduplication">

#### 3.2 In-batch synthetic de-duplication

As of [the R86 Petra][r86-release], RDP de-duplicates "synthetic duplicates" - i.e. events which
share the same event ID (`event_id`), but have different event payload (based on
`event_fingerprint`), meaning that they can be either semantically independent events (caused by
the flawed algorithms discussed above) or the same events with slightly different payloads (caused
by third-party software). For a given ETL run (batch) of events being processed,
RDB Shredder uses the following strategy:

1. Collect all the events with identical `event_id` which are left after natural-deduplication
2. Generate new random `event_id` for each of them
3. Create a [`duplicate`][duplicate-schema] context with the original `event_id` for each event
where the duplicated `event_id` was found

There is no configuration required for this functionality - de-duplication is performed
automatically in RDB Shredder, but it is highly recommended to use the
[Event Fingerprint Enrichment][fingerprint-enrichment] in order to correctly populate the
`event_fingerprint` property.

<a name="crossbatch-deduplication">

#### 3.3 Cross-batch natural de-duplication

With cross-batch natural de-duplication, we have to face a new issue: we need to track events across
multiple ETL batches to detect duplicates. We don't need to store the whole event - just the
`event_id` and the `event_fingerprint` metadata. We also need to store these in a database that
allows fast random access - we chose Amazon DynamoDB, a fully managed NoSQL database service.

##### DynamoDB table design

We store the event metadata in a DynamoDB table with the following attributes:

* `eventId`, a String
* `fingerprint`, a String
* `etlTime`, a Date
* `ttl`, a Date

A lookup into this table will tell us if the event we are looking for has been seen before based on
`event_id` and `event_fingerprint`.

We store the `etl_timestamp` to prevent issues in the case of a failed run.
If a run fails and is then rerun, we don't want the rerun to consider rows in the DynamoDB table
which were written as part of the prior failed run; otherwise all events in the rerun would be
rejected as dupes!

##### Check-and-set algorithm

It is clear as to when we need to read the event metadata from DynamoDB: during the RDB Shredder
process. But when do we write the event metadata for this run back to DynamoDB? Instead of doing all
the reads and then doing all the writes, we decided to use DynamoDB's
[conditional updates][dynamodb-cond-writes] to perform a check-and-set operation inside RDB
Shredder, on a per-event basis.

The algorithm is simple:

* Attempt to write the `event_id-event_fingerprint-etl_timestamp` triple to DynamoDB **but only if**
the `event_id-event_fingerprint` pair cannot be found with an earlier `etl_timestamp` than the
provided one
* If the write fails, we have a natural duplicate
* If the write succeeds, we know we have an event which is not a natural duplicate (it could still
be a synthetic duplicate however)

If we discover a natural duplicate, we delete it. We know that we have an "original" of this event
already safely in Redshift (because we have found it in DynamoDB).

In the code, we perform this check after we have grouped the batch by `event_id` and
`event_fingerprint`; this ensures that all check-and-set requests to a specific
`event_id-event_fingerprint` pair in DynamoDB will come from a single mapper.

##### Enabling

To enable cross-batch natural de-duplication you must provide a DynamoDB table
[configuration][dynamodb-storage-target] to EmrEtlRunner and provide
[necessary rights][dynamodb-setup-guide] in IAM. If this is not provided, then cross-batch natural
de-duplication will be disabled. In-batch de-duplication will still work however.

To avoid "cold start" problems you may want to use the [[Event-manifest-populator]] Spark job, which
backpopulates duplicate storage with events from the specified point in time.

##### Table cleanup

To make sure the DynamoDB table is not going to be overpopulated we're using
[the DynamoDB Time-to-Live][dynamodb-ttl] feature, which provides automatic cleanup after the
specified time. For event manifests this time is the etl timestamp plus 180 days and stored in the
`ttl` attribute.

##### Costs and performance penalty

Cross-batch deduplication uses DynamoDB as transient storage and therefore has associated AWS costs.
Default write capacity is 100 units, which means no matter how powerful your EMR cluster is - whole
RDB Shredder can be throttled by AWS DynamoDB. The rough cost of the default setup is 50USD per
month, however throughput can be [tweaked][dynamodb-setup-guide] according to your needs.

#### 3.4 Cross-batch synthetic de-duplication

This section hasn't been written yet.

[dynamodb-setup-guide]: https://github.com/snowplow/snowplow/wiki/Setting-up-Amazon-DynamoDB

[redshift-copy]: http://docs.aws.amazon.com/redshift/latest/dg/copy-parameters-data-source-s3.html
[dynamodb-cond-writes]: http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html#WorkingWithItems.ConditionalUpdate
[dynamodb-ttl]: http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html
[ndjson]: http://ndjson.org/
[scalding]: https://github.com/twitter/scalding
[cascading]: http://www.cascading.org/

[issue-2967]: http://snowplowanalytics.com/blog/2016/01/26/snowplow-r76-changeable-hawk-eagle-released/#deduplication

[EnrichedEvent]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/outputs/EnrichedEvent.scala
[fingerprint-enrichment]: https://github.com/snowplow/snowplow/wiki/Event-fingerprint-enrichment
[sce]: https://github.com/snowplow/snowplow/tree/master/3-enrich/scala-common-enrich
[dealing-with-duplicate-event-ids]: http://snowplowanalytics.com/blog/2015/08/19/dealing-with-duplicate-event-ids/
[r76-release]: http://snowplowanalytics.com/blog/2016/01/26/snowplow-r76-changeable-hawk-eagle-released/#deduplication
[r86-release]: http://snowplowanalytics.com/blog/2016/12/20/snowplow-r86-petra-released/
[r88-release]: http://snowplowanalytics.com/blog/2017/04/27/snowplow-r88-angkor-wat-released/
[duplicate-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/duplicate/jsonschema/1-0-0
[dynamodb-storage-target]: https://github.com/snowplow/snowplow/wiki/Configuring-storage-targets#dynamodb
