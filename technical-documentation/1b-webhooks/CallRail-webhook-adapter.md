[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > CallRail webhook adapter

## Contents

- 1 [Overview](#overview)
- 2 [Implementation](#implementation)
  - 2.1 [Event source](#source)
  - 2.2 [Snowplow adapter](#adapter)
- 3 [Events](#events)
- 4 [See also](#see-also)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track completed telephone calls logged by [CallRail][callrail-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

The [CallRail webhook API][callrail-webhooks] as of 1 November 2014.

CallRail sends events as a body- and content-type-less `POST` request with all information on the querystring.

<a name="adapter" />

### 2.2 Snowplow adapter

Implementation: [CallrailAdapter][callrail-adapter]

CallRail webhook support was implemented in [Snowplow 0.9.10][snowplow-0.9.10].

<a name="events" />

## 3. Events

All resources for this webhook's events:

| **Event**      | **JSON Schema**                                  | **JSON Paths**                                    | **Redshift Table**                                     |
|:---------------|:-------------------------------------------------|:--------------------------------------------------|:-------------------------------------------------------|
| Call complete  | [call_complete 1-0-1][call-complete-json-schema] | [call_complete_1.json][call-complete-json-paths] | [com_callrail_call_complete_1.sql][call-complete-sql] |
| Call start     | Unsupported                                      | Unsupported                                       | Unsupported                                            |

<a name="see-also" />

## 4. See also

[[CallRail webhook setup]]

[callrail-website]: http://www.callrail.com/
[callrail-webhooks]: https://support.callrail.com/hc/en-us/articles/201211133-Webhooks

[callrail-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/CallrailAdapter.scala
[snowplow-0.9.10]: https://github.com/snowplow/snowplow/releases/tag/0.9.10

[call-complete-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.callrail/call_complete/jsonschema/1-0-0
[call-complete-json-paths]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.callrail/call_complete_1.json
[call-complete-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.callrail/call_complete_1.sql
