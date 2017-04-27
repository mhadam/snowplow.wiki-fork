[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](SnowPlow-technical-documentation) > [**Webhooks**](Webhooks) > Iglu webhook adapter

## Contents

- 1. [Overview](#overview)  
- 2. [Implementation](#implementation)  
  - 2.1 [Event source](#source)  
  - 2.2 [Example](#example)  
  - 2.3 [Snowplow adapter](#adapter)  
- 3. [Events](#events)  
- 4. [See also](#see-also)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track events sent via `GET` or `POST` requests containing an [Iglu][iglu]-compatible event payload.

You can use this adapter with vendors who allow you define your own event types for "postback". An example of a vendor who does this is [AD-X Tracking][adxtracking-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

This adapter assumes that events are sent via a `GET` request with all information on the querystring.

The querystring's name-value pairs **must** include a `schema` parameter, which is set to a valid Iglu self-describing schema URI, such as:

```
iglu:com.acme.postbacks/install_error/jsonschema/1-0-0
```

<a name="example" />

### 2.2 Example

Here is an example of an Iglu-compatible event sent as a `GET` request:

```
http://snplow.acme.com/com.snowplowanalytics.iglu/v1?schema=iglu%3Acom.acme%2Fcampaign%2Fjsonschema%2F1-0-0
  &user=6353af9b-e288-4cf3-9f1c-b377a9c84dac&name=download&publisher_name=Organic&source=&tracking_id=&ad_unit=
```

This will be converted by the Iglu webhook adapter into a self-describing JSON looking like this:

```json
{
  "schema":"iglu:com.acme/campaign/jsonschema/1-0-0",
  "data": {
    "name":"download",
    "source":null,
    "ad_unit":null,
    "tracking_id":null,
    "publisher_name":"Organic",
    "user":"6353af9b-e288-4cf3-9f1c-b377a9c84dac"
}
```

An example as a POST request:

```
http://snplow.acme.com/com.snowplowanalytics.iglu/v1 -d '{
  "schema":"iglu:com.acme/campaign/jsonschema/1-0-0",
  "data": {
    "name":"download",
    "source":null,
    "ad_unit":null,
    "tracking_id":null,
    "publisher_name":"Organic",
    "user":"6353af9b-e288-4cf3-9f1c-b377a9c84dac"
  }
}'
```

Note that successful processing through into Amazon Redshift will depend on the appropriate JSON Schema, JSON Paths file and Redshift table definition all being defined correctly.

<a name="adapter" />

### 2.3 Snowplow adapter

Implementation: [IgluAdapter][iglu-adapter]

Iglu webhook support was implemented in [Snowplow 0.9.11][snowplow-0.9.11].

Iglu webhook POST support was implemented in [Snowplow R83 Bald Eagle][snowplow-r83].

<a name="events" />

## 3. Events

This webhook adapter supports any event, as long as it is a valid [Iglu][iglu] self-describing event.

Note that a limitation of this adapter with GET requests is that all event properties will end up being "stringly typed" - the adapter has no way of knowing which parameters should be converted into numbers, booleans, date-times or similar.

<a name="see-also" />

## 4. See also

[[Iglu webhook setup]]

[adxtracking-website]: http://adxtracking.com/

[iglu]: https://github.com/snowplow/iglu
[iglu-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/IgluAdapter.scala
[snowplow-0.9.11]: https://github.com/snowplow/snowplow/releases/tag/0.9.11
[snowplow-r83]: https://github.com/snowplow/snowplow/releases/tag/r83-bald-eagle
