<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > Pagerduty webhook adapter

## Contents

- 1 [Overview](#overview)
- 2 [Implementation](#implementation)
  - 2.1 [Event source](#source)
  - 2.2 [Snowplow adapter](#adapter)
- 3 [Events](#events)
- 4 [See also](#see-also)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track a variety of events logged by [PagerDuty][pagerduty-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

Details of the PagerDuty webhook format as of 1 November 2014.

PagerDuty sends multiple events together as a `POST` request with all information in the body, with `application/json` as the content type.

<a name="adapter" />

### 2.2 Snowplow adapter

Implementation: [PagerdutyAdapter][pagerduty-adapter]

PagerDuty webhook support was implemented in [Snowplow 0.9.13][snowplow-0.9.13].

<a name="events" />

## 3. Events

All resources for this webhook's events:

| **Event**      | **JSON Schema**                                  | **JSON Paths**                                    | **Redshift Table**                                     |
|:---------------|:-------------------------------------------------|:--------------------------------------------------|:-------------------------------------------------------|
| Incident       | [incident 1-0-0][incident-json-schema]          | [incident_1.json][incident-json-paths]           | [com_pagerduty_incident_1.sql][incident-sql]          |

While seven potential events can be sent from PagerDuty; as each of them has a nearly identical data structure they have been melded into one table dubbed **Incident**.

<a name="see-also" />

## 4. See also

[[Pagerduty webhook setup]]

[pagerduty-website]: http://www.pagerduty.com/
[pagerduty-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/PagerdutyAdapter.scala
[snowplow-0.9.13]: https://github.com/snowplow/snowplow/releases/tag/0.9.13

[incident-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.pagerduty/incident/jsonschema/1-0-0
[incident-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.pagerduty/incident_1.json
[incident-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.pagerduty/incident_1.sql
