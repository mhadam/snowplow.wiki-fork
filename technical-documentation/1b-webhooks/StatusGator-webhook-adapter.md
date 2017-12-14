<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > StatusGator webhook adapter

## Contents

- 1 [Overview](#overview)
- 2 [Implementation](#implementation)
  - 2.1 [Event source](#source)
  - 2.2 [Snowplow adapter](#adapter)
- 3 [Events](#events)
- 4 [See also](#see-also)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track status change events logged by [StatusGator][statusgator-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

Details of the StatusGator webhook format as of 1 November 2017.

StatusGator sends single events together as a `POST` request with all information in the body, with `application/x-www-form-urlencoded` as the content type.

<a name="adapter" />

### 2.2 Snowplow adapter

Implementation: [StatusGatorAdapter][statusgator-adapter]

StatusGator webhook support was implemented in [R97 Knossos][r97].

<a name="events" />

## 3. Events

All resources for this webhook's events:

| **Event**      | **JSON Schema**                                  | **JSON Paths**                                    | **Redshift Table**                                      |
|:---------------|:-------------------------------------------------|:--------------------------------------------------|:--------------------------------------------------------|
| statuse Change | [status_change 1-0-0][status-change-json-schema] | [status_change_1.json][status-change-json-paths]  | [com_statusgator_status_change_1.sql][status-change-sql] |

<a name="see-also" />

## 4. See also

[[StatusGator webhook setup]]

[statusgator-website]: https://statusgator.com/
[statusgator-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/StatusGatorAdapter.scala
[r97]: https://github.com/snowplow/snowplow/releases/tag/r97-knossos

[status-change-json-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.statusgator/status_change/jsonschema/1-0-0

[status-change-json-paths]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.statusgator/status_change_1.json

[status-change-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.statusgator/status_change_1.sql
