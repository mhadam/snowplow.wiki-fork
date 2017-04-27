<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Snowplow Analytics SDK**](Snowplow-Analytics-SDK) » Python Analytics SDK Event Transformer

## Contents

- 1 [Overview](#overview)  
- 2 [The JSON Event Transformer](#transformer)  
- 3 [Example](#example)  

<a name="overview" />

## Overview

The Snowplow enriched event is a relatively complex TSV string containing self-describing JSONs. Rather than work with this structure directly, Snowplow analytics SDKs ship with *event transformers*, which translate the Snowplow enriched event format into something more convenient for engineers and analysts.
Rather than work with this structure directly, Snowplow analytics SDKs ship with *event transformers*, which translate the Snowplow enriched event format into something more convenient for engineers and analysts.

As the Snowplow enriched event format evolves towards a cleaner [Apache Avro](https://avro.apache.org/)-based structure, we will be updating this Analytics SDK to maintain compatibility across different enriched event versions.

Working with the Snowplow Python Analytics SDK therefore has two major advantages over working with Snowplow enriched events directly:

1. The SDK reduces your development time by providing analyst- and developer-friendly transformations of the Snowplow enriched event format
2. The SDK futureproofs your code against new releases of Snowplow which update our enriched event format

Currently the Python Analytics SDK ships with one event transformer: the JSON Event Transformer. 

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

<a name="example" />

## Example

You can convert an enriched event TSV string to a JSON like this:

```python
import snowplow_analytics_sdk.event_transformer
import snowplow_analytics_sdk.snowplow_event_transformation_exception

try:
    print(snowplow_analytics_sdk.event_transformer.transform(my_enriched_event_tsv))
except snowplow_analytics_sdk.snowplow_event_transformation_exception.SnowplowEventTransformationException as e:
    for error_message in e.error_messages:
        print(error_message)
```

Note that if the transformation throws an exception, the `error_messages` field of that exception will contain information about everything which was invalid about the TSV.

[Back to top](#top)  
[Back to Python Analytics SDK contents][contents]

[contents]: Python-Analytics-SDK
