<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > [[Unbounce webhook adapter]]

## Contents

- 1 [Overview](#overview)
- 2 [Implementation](#implementation)
  - 2.1 [Event source](#source)
  - 2.2 [Snowplow adapter](#adapter)
- 3 [Events](#events)
- 4 [See also](#see-also)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track a variety of events logged by [Unbounce][unbounce-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

Details of the Unbounce webhook format as of 15.12.2017.

Unbounce sends events as a `POST` request with all information in the body, with `application/x-www-form-urlencoded` as the content type.

<a name="adapter" />

### 2.2 Snowplow adapter

Implementation: [UnbounceAdapter][unbounce-adapter]

Unbounce webhook support was implemented in [R97 Knossos][r97].

<a name="events" />

## 3. Events

All resources for this webhook's events:

| **Event**      | **JSON Schema**                                  | **JSON Paths**                                    | **Redshift Table**                                     |
|:---------------|:-------------------------------------------------|:--------------------------------------------------|:-------------------------------------------------------|
|Form Submit     | |[com_unbounce_form_post 1-0-0][com_unbounce_offline_message_1-schema]               | [com_unbounce_form_post_1.json][com_mailgun_recipient_unsubscribed-json]               | [com_unbounce_form_post_1.sql] [com_unbounce_form_post_1-sql]               |


<a name="see-also" />

## 4. See also

[[Unbounce webhook setup]]

[unbounce-website]: https://unbounce.com/
[unbounce-webhooks]: https://documentation.unbounce.com/hc/en-us/articles/203510044-Using-a-Webhook
[r97]: https://github.com/snowplow/snowplow/releases/tag/r97-knossos

[unbounce-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/UnbounceAdapter.scala

[com_unbounce_offline_message_1-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.unbounce/form_post/jsonschema/1-0-0

[com_unbounce_offline_message_1-json]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.unbounce/form_post_1.json

[com_unbounce_offline_message_1-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.unbounce/form_post_1.sql

