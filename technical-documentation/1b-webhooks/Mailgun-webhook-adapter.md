<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > [[Mailgun webhook adapter]]

## Contents

- 1 [Overview](#overview)
- 2 [Implementation](#implementation)
  - 2.1 [Event source](#source)
  - 2.2 [Snowplow adapter](#adapter)
- 3 [Events](#events)
- 4 [See also](#see-also)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track a variety of events logged by [Mailgun][mailgun-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

Details of the Mailgun webhook format as of 15.12.2017.

Mailgun events as a `POST` request with all information in the body, with `application/x-www-form-urlencoded` or `multipart/form-data` as the content type.

<a name="adapter" />

### 2.2 Snowplow adapter

Implementation: [MailgunAdapter][mailgun-adapter]

Mailgun webhook support was implemented in [R97 Knossos][r97].

<a name="events" />

## 3. Events

All resources for this webhook's events:

| **Event**      | **JSON Schema**                                  | **JSON Paths**                                    | **Redshift Table**                                     |
|:---------------|:-------------------------------------------------|:--------------------------------------------------|:-------------------------------------------------------|
|Hard Bounces       | |[com_mailgun_message_bounced 1-0-0][com_mailgun_message_bounced-schema]               | [com_mailgun_message_bounced_1.json][com_mailgun_message_bounced-json]               | [com_mailgun_message_bounced_1.sql] [com_mailgun_message_bounced-sql]               |
|Clicks             | |[com_mailgun_message_clicked 1-0-0][com_mailgun_message_clicked-schema]               | [com_mailgun_message_clicked_1.json][com_mailgun_message_clicked-json]               | [com_mailgun_message_clicked_1.sql] [com_mailgun_message_clicked-sql]               |
|Spam Complaints    | |[com_mailgun_message_complained 1-0-0][com_mailgun_message_complained-schema]         | [com_mailgun_message_complained_1.json][com_mailgun_message_complained-json]         | [com_mailgun_message_complained_1.sql] [com_mailgun_message_complained-sql]         |
|Delivered Messages | |[com_mailgun_message_delivered 1-0-0][com_mailgun_message_delivered-schema]           | [com_mailgun_message_delivered_1.json][com_mailgun_message_delivered-json]           | [com_mailgun_message_delivered_1.sql] [com_mailgun_message_delivered-sql]           |
|Dropped Messages   | |[com_mailgun_message_dropped 1-0-0][com_mailgun_message_dropped-schema]               | [com_mailgun_message_dropped_1.json][com_mailgun_message_dropped-json]               | [com_mailgun_message_dropped_1.sql] [com_mailgun_message_dropped-sql]               |
|Opens              | |[com_mailgun_message_opened 1-0-0][com_mailgun_message_opened-schema]                 | [com_mailgun_message_opened_1.json][com_mailgun_message_opened-json]                 | [com_mailgun_message_opened_1.sql] [com_mailgun_message_opened-sql]                 |
|Unsubscribes       | |[com_mailgun_recipient_unsubscribed 1-0-0][com_mailgun_recipient_unsubscribed-schema] | [com_mailgun_recipient_unsubscribed_1.json][com_mailgun_recipient_unsubscribed-json] | [com_mailgun_recipient_unsubscribed_1.sql] [com_mailgun_recipient_unsubscribed-sql] |


<a name="see-also" />

## 4. See also

[[Mailgun webhook setup]]

[mailgun-website]: https://www.mailgun.com/
[mailgun-webhooks]: https://documentation.mailgun.com/en/latest/user_manual.html#webhooks
[r97]: https://github.com/snowplow/snowplow/releases/tag/r97-knossos
[mailgun-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/MailgunAdapter.scala


[com_mailgun_message_bounced-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.mailgun/message_bounced/jsonschema/1-0-0
[com_mailgun_message_clicked-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.mailgun/message_clicked/jsonschema/1-0-0
[com_mailgun_message_complained-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.mailgun/message_complained/jsonschema/1-0-0
[com_mailgun_message_delivered-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.mailgun/message_delivered/jsonschema/1-0-0
[com_mailgun_message_dropped-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.mailgun/message_dropped/jsonschema/1-0-0
[com_mailgun_message_opened-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.mailgun/message_opened/jsonschema/1-0-0
[com_mailgun_recipient_unsubscribed-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.mailgun/recipient_unsubscribed/jsonschema/1-0-0

[com_mailgun_message_bounced-json]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailgun/message_bounced_1.json
[com_mailgun_message_clicked-json]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailgun/message_clicked_1.json
[com_mailgun_message_complained-json]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailgun/message_complained_1.json
[com_mailgun_message_delivered-json]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailgun/message_delivered_1.json
[com_mailgun_message_dropped-json]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailgun/message_dropped_1.json
[com_mailgun_message_opened-json]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailgun/message_opened_1.json
[com_mailgun_recipient_unsubscribed-json]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailgun/recipient_unsubscribed_1.json

[com_mailgun_message_bounced-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailgun/message_clicked_1.sql
[com_mailgun_message_clicked-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailgun/message_dropped_1.sql
[com_mailgun_message_complained-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailgun/message_complained_1.sql
[com_mailgun_message_delivered-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailgun/recipient_unsubscribed_1.sql
[com_mailgun_message_dropped-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailgun/message_opened_1.sql
[com_mailgun_message_opened-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailgun/message_bounced_1.sql
[com_mailgun_recipient_unsubscribed-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailgun/message_delivered_1.sql
