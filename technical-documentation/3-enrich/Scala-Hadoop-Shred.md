[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow technical documentation) > [**Enrichment**](Enrichment) > Scala Hadoop Shred

### 1. Overview

Scala Hadoop Shred is Hadoop job, written in [Scalding][scalding] (Scala API for [Cascading][cascading]) and allowing you to split (shred) Snowplow enriched event, produced by Scala Hadoop Enrich into separate enrities.
Scala Hadoop Shred utilizes the [scala-common-enrich][sce] Scala project to load enriched events.

You will typically run the Scala Hadoop Shred jar as part of an EMR jobflow, started by [EmrEtlRunner](EmrEtlRunner).
It is designed to be used downstream of the Scala Hadoop Enrich and upstream of [StorageLoader](StorageLoader).

Scala Hadoop Shred has two primary tasks:

1. Shred enriched event into `atomic-event` TSV and associated JSONs
2. Make `event_id`s for all events unique

### 2. Shredding

Snowplow [enriched event][EnrichedEvent] is 131-column TSV file, produced by Scala Hadoop Enrich. 
Each line contains all information about specific events, including id, timestamps, custom and derived contexts and many more.

Shredding is process of splitting `EnrichedEvent` TSV into following parts:

1. **Atomic event**. TSV line very similar to `EnrichedEvent` but not contining
   JSON fields (`contexts`, `derived_contexts` and `unstruct_event`). Result
   will be stored in path similar to `shredded/good/run=2016-11-26-21-48-42/atomic-events/part-00000`
   and wil be available to load via [StorageLoader](StorageLoader) or directly
   via Redshift [COPY][redshift-copy].
2. **Contexts**. This part consists of two extracted above JSON fields: 
   `contexts` and `derived_contexts`, which are validated (on enrichment-step)
   self-describing JSONs. But unlike usual self-describing JSONs consisting of
   `schema` string and `data` object, these ones consist of `schema` object
   (like in JSON Schema), usual `data` object and `hierarchy` object. This 
   `hierarchy` contains data to later join your contexts SQL tables with 
   `atomic.events` table. Result will be stored in path similar to 
   `shredded/good/run=2016-11-26-21-48-42/com.acme/mycontext/jsonschema/1-0-1/part-00000`,
   where files like `part-00000` are valid NDJSONs and will be available to load
   via [StorageLoader](StorageLoader) or directly via Redshift [COPY][redshift-copy].
3. **Self-describing (unstructured) event**. Pretty much like contexts this is
   same JSON with `schema`, `data` and `hierarchy` fields. The only difference
   is that it has one-to-one relation to `atomic.events`, whereas contexts have
   many-to-one relation.

Shredding is classic example of Hadoop [mapper](https://hadoop.apache.org/docs/r2.6.2/api/org/apache/hadoop/mapreduce/Mapper.html) - 
each line (event) is independent of each other, it is a function which has 
single input and output.

More details on what shredding is can be found on dedicated 
[shredding](Shredding) page.

### 3. Deduplication

Duplicates is common problem in event pipelines, it is described 
[many][dealing-with-duplicate-event-ids] [times][r76-release]. Basically
problem is that we cannot guarantee that every event has unique `UUID` because

1. we have no exactly-once-delivery guarantee 
2. some user-side software makes events send more than once
3. flawed [algorithms][issue-2967]

There are four strategies planned for Scala Hadoop Shred's deduplication:

| Strategy                             | Batch?      | Same event ID? | Same event fingerprint? | Availability                              |
|--------------------------------------|-------------|----------------|-------------------------|-------------------------------------------|
| In-batch natural de-duplication      | In-batch    | Yes            | Yes                     | [R76 Changeable Hawk-Eagle] [r76-release] |
| In-batch synthetic de-duplication    | In-batch    | Yes            | No                      | R86 Petra                                 |
| Cross-batch natural de-duplication   | Cross-batch | Yes            | Yes                     | Planned                                   |
| Cross-batch synthetic de-duplication | Cross-batch | Yes            | No                      | Planned                                   |

We will cover these in turn:

#### 3.1 In-batch natural de-duplication

If `event_id` and `event_fingerprint` (hash calculated from payload fields)
   for two events are identical - drop all these events except first one. This
   removes natural duplicates.

**ANTON TO UPDATE THIS WITH MORE DETAIL FROM THE R76 RELEASE POST**

#### 3.2 In-batch synthetic de-duplication

For all events left having same `event_id` - regenerate random
   `event_id` and add new `duplicate` context containing original `event_id`.

**ANTON TO UPDATE THIS WITH MORE DETAIL.**

#### 3.3 Cross-batch natural de-duplication

**ALEX TO ADD**

#### 3.4 Cross-batch synthetic de-duplication

**ALEX TO ADD**

[redshift-copy]: http://docs.aws.amazon.com/redshift/latest/dg/copy-parameters-data-source-s3.html
[ndjson]: http://ndjson.org/
[scalding]: https://github.com/twitter/scalding
[cascading]: http://www.cascading.org/

[issue-2967]: http://snowplowanalytics.com/blog/2016/01/26/snowplow-r76-changeable-hawk-eagle-released/#deduplication

[EnrichedEvent]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/outputs/EnrichedEvent.scala
[sce]: https://github.com/snowplow/snowplow/tree/master/3-enrich/scala-common-enrich
[dealing-with-duplicate-event-ids]: http://snowplowanalytics.com/blog/2015/08/19/dealing-with-duplicate-event-ids/
[r76-release]: http://snowplowanalytics.com/blog/2016/01/26/snowplow-r76-changeable-hawk-eagle-released/#deduplication
