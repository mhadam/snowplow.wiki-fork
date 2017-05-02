<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > SendGrid webhook adapter

## Contents

- 1 [Overview](#overview)
- 2 [Implementation](#implementation)
  - 2.1 [Event source](#source)
  - 2.2 [Snowplow adapter](#adapter)
- 3 [Events](#events)
- 4 [See also](#see-also)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track a variety of events logged by [SendGrid][sendgrid-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

Details of the SendGrid webhook format as of 12th November 2015.

SendGrid sends events as a `POST` request with all information in the body, with `application/json` as the content type.

<a name="adapter" />

### 2.2 Snowplow adapter

Implementation: [SendGrid][sendgrid-adapter]

SendGrid webhook support was implemented in [Snowplow 75 Long-Legged Buzzard][snowplow-release].

<a name="events" />

## 3. Events

All resources for this webhook's events:

| **Event**         | **JSON Schema**                                            | **JSON Paths**                                             | **Redshift Table**                                            |
|:------------------|:-----------------------------------------------------------|:-----------------------------------------------------------|:--------------------------------------------------------------|
| Processed         | [processed 1-0-0][processed-json-schema]                  | [processed_1.json][processed-json-paths]                  | [com_sendgrid_processed_1.sql][processed-sql]                |
| Dropped           | [dropped 1-0-0][dropped-json-schema]                      | [dropped_1.json][dropped-json-paths]                      | [com_sendgrid_dropped_1.sql][dropped-sql]                    |
| Delivered         | [delivered 1-0-0][delivered-json-schema]                  | [delivered_1.json][delivered-json-paths]                  | [com_sendgrid_delivered_1.sql][delivered-sql]                |
| Deferred          | [deferred 1-0-0][deferred-json-schema]                    | [deferred_1.json][deferred-json-paths]                    | [com_sendgrid_deferred_1.sql][deferred-sql]                  |
| Bounce            | [bounce 1-0-0][bounce-json-schema]                        | [bounce_1.json][bounce-json-paths]                        | [com_sendgrgid_bounce_1.sql][bounce-sql]                     |
| Open              | [open 1-0-0][open-json-schema]                            | [open_1.json][open-json-paths]                            | [com_sendgrid_open_1.sql][open-sql]                          |
| Click             | [click 1-0-0][click-json-schema]                          | [click_1.json][click-json-paths]                          | [com_sendgrid_click_1.sql][click-sql]                        |
| Spam Report       | [spam_report 1-0-0][spam_report-json-schema]              | [spam_report_1.json][spam_report-json-paths]              | [com_sendgrid_spam_report_1.sql][spam_report-sql]            |
| Unsubscribe       | [unsubscribe 1-0-0][unsubscribe-json-schema]              | [unsubscribe_1.json][unsubscribe-json-paths]              | [com_sendgrid_unsubscribe_1.sql][unsubscribe-sql]            |
| Group Unsubscribe | [group_unsubscribe 1-0-0][group_unsubscribe-json-schema]  | [group_unsubscribe_1.json][group_unsubscribe-json-paths]  | [com_sendgrid_group_unsubscribe_1.sql][group_unsubscribe-sql]|
| Group Resubscribe | [group_resubscribe 1-0-0][group_resubscribe-json-schema]  | [group_resubscribe_1.json][group_resubscribe-json-paths]  | [com_sendgrid_resubscribe_1.sql][group_resubscribe-sql]      |


<a name="see-also" />

## 4. See also

[[SendGrid webhook setup]]

[sendgrid-website]: http://sendgrid.com/

[sendgrid-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/SendgridAdapter.scala
[snowplow-release]: https://github.com/snowplow/snowplow/releases/tag/r75-long-legged-buzzard

[processed-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/processed/jsonschema/1-0-0
[processed-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/processed_1.json
[processed-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/processed_1.sql

[dropped-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/dropped/jsonschema/1-0-0
[dropped-json-paths]: https://github.com/snowplow/snowplow/tre0.9.11e/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/dropped_1.json
[dropped-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/dropped_1.sql

[delivered-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/delivered/jsonschema/1-0-0
[delivered-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/delivered.json
[delivered-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/delivered.sql

[deferred-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/deferred/jsonschema/1-0-0
[deferred-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/deferred_1.json
[deferred-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/deferred_1.sql

[bounce-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/bounce/jsonschema/1-0-0
[bounce-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/bounce_1.json
[bounce-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/bounce_1.sql

[open-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/open/jsonschema/1-0-0
[open-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/open_1.json
[open-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/open_1.sql

[click-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/click/jsonschema/1-0-0
[click-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/click_1.json
[click-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/click_1.sql

[spam_report-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/spam_report/jsonschema/1-0-0
[spam_report-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/spam_report_1.json
[spam_report-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/spam_report_1.sql

[unsubscribe-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/unsubscribe/jsonschema/1-0-0
[unsubscribe-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/unsubscribe_1.json
[unsubscribe-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/unsubscribe_1.sql

[unsubscribe-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/unsubscribe/jsonschema/1-0-0
[unsubscribe-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/unsubscribe_1.json
[unsubscribe-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/unsubscribe_1.sql

[group_unsubscribe-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/group_unsubscribe/jsonschema/1-0-0
[group_unsubscribe-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/group_unsubscribe_1.json
[group_unsubscribe-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/group_unsubscribe_1.sql

[group_resubscribe-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.sendgrid/group_resubscribe/jsonschema/1-0-0
[group_resubscribe-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.sendgrid/group_resubscribe_1.json
[group_resubscribe-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.sendgrid/group_resubscribe_1.sql
