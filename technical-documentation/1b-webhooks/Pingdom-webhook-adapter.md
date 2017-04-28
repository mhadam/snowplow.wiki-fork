<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > Pingdom webhook adapter

## Contents

- 1. [Overview](#overview)
- 2. [Implementation](#implementation)
  - 2.1 [Event source](#source)
  - 2.2 [Snowplow adapter](#adapter)
- 3. [Events](#events)
- 4. [See also](#see-also)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track a variety of events logged by [Pingdom][pingdom-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

Details of the Pingdom webhook format as of 1 November 2014.

Pingdom sends a single event as a `GET` request with all information in the querystring, without any content type.

<a name="adapter" />

### 2.2 Snowplow adapter

Implementation: [PingdomAdapter][pingdom-adapter]

Pingdom webhook support was implemented in [Snowplow 0.9.13][snowplow-0.9.13].

<a name="events" />

## 3. Events

All resources for this webhook's events:

| **Event**      | **JSON Schema**                                  | **JSON Paths**                                    | **Redshift Table**                                     |
|:---------------|:-------------------------------------------------|:--------------------------------------------------|:-------------------------------------------------------|
| Assign          | [incident_assign 1-0-0][assign-json-schema]         | [incident_assign_1.json][assign-json-paths]         | [com_pingdom_incident_assign_1.sql][assign-sql]         |
| Notify of close | [incident_notify_of_close 1-0-0][close-json-schema] | [incident_notify_of_close_1.json][close-json-paths] | [com_pingdom_incident_notify_of_close_1.sql][close-sql] |

<a name="see-also" />

## 4. See also

[[Pingdom webhook setup]]

[pingdom-website]: https://www.pingdom.com/
[pingdom-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/PingdomAdapter.scala
[snowplow-0.9.13]: https://github.com/snowplow/snowplow/releases/tag/0.9.13

[assign-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.pingdom/incident_assign/jsonschema/1-0-0
[assign-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.pingdom/incident_assign_1.json
[assign-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.pingdom/incident_assign_1.sql

[close-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.pingdom/incident_notify_of_close/jsonschema/1-0-0
[close-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.pingdom/incident_notify_of_close_1.json
[close-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.pingdom/incident_notify_of_close_1.sql
