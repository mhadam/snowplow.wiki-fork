<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Snowplow Analytics SDK**](Snowplow-Analytics-SDK) » Scala Analytics SDK

- 1 [Overview](#overview)  
- 2 [The JSON Event Transformer](#transformer)  
- 3 [Examples](#example)  
  - 3.1 [Using from Apache Spark](#spark)  
  - 3.2 [Using from AWS Lambda](#lambda)  

<a name="overview" />

## Overview

The Snowplow enriched event is a relatively complex TSV string containing self-describing JSONs. 
Rather than work with this structure directly, Snowplow analytics SDKs ship with *event transformers*, which translate the Snowplow enriched event format into something more convenient for engineers and analysts.

As the Snowplow enriched event format evolves towards a cleaner [Apache Avro](https://avro.apache.org/)-based structure, we will be updating this Analytics SDK to maintain compatibility across different enriched event versions.

Working with the Snowplow Scala Analytics SDK therefore has two major advantages over working with Snowplow enriched events directly:

1. The SDK reduces your development time by providing analyst- and developer-friendly transformations of the Snowplow enriched event format
2. The SDK futureproofs your code against new releases of Snowplow which update our enriched event format

Currently the Analytics SDK for Scala ships with one event transformer: the JSON Event Transformer. 

<a name="transformer" />

## The JSON Event Transformer

The JSON Event Transformer takes a Snowplow enriched event and converts it into a JSON ready for further processing. This transformer was adapted from the code used to load Snowplow events into Elasticsearch in the Kinesis real-time pipeline.

The JSON Event Transformer converts a Snowplow enriched event into a single JSON like so:

```json
{ "app_id":"demo",
  "platform":"web","etl_tstamp":"2015-12-01T08:32:35.048Z",
  "collector_tstamp":"2015-12-01T04:00:54.000Z","dvce_tstamp":"2015-12-01T03:57:08.986Z",
  "event":"page_view","event_id":"f4b8dd3c-85ef-4c42-9207-11ef61b2a46e","txn_id":null,
  "name_tracker":"co","v_tracker":"js-2.5.0","v_collector":"clj-1.0.0-tom-0.2.0",...
```

The most complex piece of processing is the handling of the self-describing JSONs found in the enriched event's `unstruct_event`, `contexts` and `derived_contexts` fields. All self-describing JSONs found in the event are flattened into top-level plain (i.e. not self-describing) objects within the enriched event JSON.

For example, if an enriched event contained a `com.snowplowanalytics.snowplow/link_click/jsonschema/1-0-1`, then the final JSON would contain:

```json
{ "app_id":"demo","platform":"web","etl_tstamp":"2015-12-01T08:32:35.048Z",
  "unstruct_event_com_snowplowanalytics_snowplow_link_click_1": {
    "targetUrl":"http://www.example.com",
    "elementClasses":["foreground"],
    "elementId":"exampleLink"
  },...
```

`EventTransformer` has three primary functions:

* `transform` - simply accepts enriched event as a string and returns JSON (as `String`) of above structure as a result
* `transformWithInventory` - accepts enriched event as a string and returns JSON  (as `String`) of above structure as a result **and** event's inventory (explained below)
* `jsonifyGoodEvent` - same as above, but accepts already splitted TSV columns (as `Array[String]`) and returns JSON as json4s `JObject` in case you'll need to perform some modifications with this AST

### Event inventory

Invetory is simply a metadata about shredded type:

1. Its Iglu URI (e.g. `iglu:com.acme/context/jsonschema/1-0-0`)
2. Where it was extracted from: `unstruct_event` column (`UnstructEvent`), `contexts` column (`Contexts(CustomContexts)`) or `derived_contexts` column (`Contexts(DerivedContexts)`)

<a name="example" />

## Examples

<a name="spark" />

### Using from Apache Spark

The Scala Analytics SDK is a great fit for performing Snowplow [event data modeling](http://snowplowanalytics.com/blog/2016/03/16/introduction-to-event-data-modeling/) in Apache Spark and Spark Streaming.

Here’s the code we use internally for our own data modeling jobs:

```scala
import com.snowplowanalytics.snowplow.analytics.scalasdk.json.EventTransformer

val events = input
  .map(line => EventTransformer.transform(line))
  .filter(_.isRight)
  .flatMap(_.right.toOption)

val dataframe = ctx.read.json(events)
```

<a name="lambda" />

### Using from AWS Lambda

The Scala Analytics SDK is a great fit for performing analytics-on-write, monitoring or alerting on Snowplow event streams using AWS Lambda.

Here’s some sample code for transforming enriched events into JSON inside a Scala Lambda:

```scala
import com.snowplowanalytics.snowplow.analytics.scalasdk.json.EventTransformer

def recordHandler(event: KinesisEvent) {

  val events = for {
    rec <- event.getRecords
    line = new String(rec.getKinesis.getData.array())
    json = EventTransformer.transform(line)
  } yield json
```

[Back to top](#top)  
[Back to Scala Analytics SDK contents][contents]

[contents]: Scala-Analytics-SDK
