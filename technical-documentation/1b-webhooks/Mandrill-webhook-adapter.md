<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > Mandrill webhook adapter

## Contents

- 1 [Overview](#overview)
- 2 [Implementation](#implementation)
  - 2.1 [Event source](#source)
  - 2.2 [Snowplow adapter](#adapter)
- 3 [Events](#events)
- 4 [See also](#see-also)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track a variety of events logged by [Mandrill][mandrill-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

Details of the Mandrill webhook format as of 1 November 2014.

Mandrill sends multiple events together as a `POST` request with all information in the body, with `application/x-www-form-urlencoded` as the content type.

<a name="adapter" />

### 2.2 Snowplow adapter

Implementation: [MandrillAdapter][mandrill-adapter]

Mandrill webhook support was implemented in [Snowplow 0.9.14][snowplow-0.9.14].

<a name="events" />

## 3. Events

All resources for this webhook's events:

| **Event**      | **JSON Schema**                                  | **JSON Paths**                                    | **Redshift Table**                                     |
|:---------------|:-------------------------------------------------|:--------------------------------------------------|:-------------------------------------------------------|
| Sent                   | [message_sent 1-0-0][sent-json-schema]                 | [message_sent_1.json][sent-json-paths]                 | [com_mandrill_message_sent_1.sql][sent-sql]                 |
| Hard Bounce            | [message_bounced 1-0-0][bounced-json-schema]           | [message_bounced_1.json][bounced-json-paths]           | [com_mandrill_message_bounced_1.sql][bounced-sql]           |
| Soft Bounce            | [message_soft_bounced 1-0-0][soft-bounced-json-schema] | [message_soft_bounced_1.json][soft-bounced-json-paths] | [com_mandrill_message_soft_bounced_1.sql][soft-bounced-sql] |
| Opened                 | [message_opened 1-0-0][opened-json-schema]             | [message_opened_1.json][opened-json-paths]             | [com_mandrill_message_opened_1.sql][opened-sql]             |
| Marked as Spam         | [message_marked_as_spam 1-0-0][spam-json-schema]            | [message_marked_as_spam_1.json][spam-json-paths]       | [com_mandrill_message_marked_as_spam.sql][spam-sql]         |
| Rejected               | [message_rejected 1-0-0][rejected-json-schema]         | [message_rejected_1.json][rejected-json-paths]         | [com_mandrill_message_rejected_1.sql][rejected-sql]         |
| Delayed                | [message_delayed 1-0-0][delayed-json-schema]           | [message_delayed_1.json][delayed-json-paths]           | [com_mandrill_message_delayed_1.sql][delayed-sql]           |
| Clicked                | [message_clicked 1-0-0][clicked-json-schema]           | [message_clicked_1.json][clicked-json-paths]           | [com_mandrill_message_clicked_1.sql][clicked-sql]           |
| Recipient Unsubscribed | [recipient_unsubscribed 1-0-0][unsub-json-schema]      | [recipient_unsubscribed_1.json][unsub-json-paths]      | [com_mandrill_recipient_unsubscribed_1.sql][unsub-sql]      |

<a name="see-also" />

## 4. See also

[[Mandrill webhook setup]]

[mandrill-website]: https://mandrill.com/
[mandrill-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/MandrillAdapter.scala
[snowplow-0.9.14]: https://github.com/snowplow/snowplow/releases/tag/0.9.14

[sent-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mandrill/message_sent/jsonschema/1-0-0
[bounced-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mandrill/message_bounced/jsonschema/1-0-0
[soft-bounced-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mandrill/message_soft_bounced/jsonschema/1-0-0
[opened-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mandrill/message_opened/jsonschema/1-0-0
[spam-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mandrill/message_marked_as_spam/jsonschema/1-0-0
[rejected-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mandrill/message_rejected/jsonschema/1-0-0
[delayed-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mandrill/message_delayed/jsonschema/1-0-0
[clicked-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mandrill/message_clicked/jsonschema/1-0-0
[unsub-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mandrill/recipient_unsubscribed/jsonschema/1-0-0

[sent-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.mandrill/message_sent_1.json
[bounced-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.mandrill/message_bounced_1.json
[soft-bounced-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.mandrill/message_soft_bounced_1.json
[opened-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.mandrill/message_opened_1.json
[spam-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.mandrill/message_marked_as_spam_1.json
[rejected-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.mandrill/message_rejected_1.json
[delayed-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.mandrill/message_delayed_1.json
[clicked-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.mandrill/message_clicked_1.json
[unsub-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.mandrill/recipient_unsubscribed_1.json

[sent-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.mandrill/message_sent_1.sql
[bounced-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.mandrill/message_bounced_1.sql
[soft-bounced-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.mandrill/message_soft_bounced_1.sql
[opened-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.mandrill/message_opened_1.sql
[spam-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.mandrill/message_marked_as_spam_1.sql
[rejected-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.mandrill/message_rejected_1.sql
[delayed-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.mandrill/message_delayed_1.sql
[clicked-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.mandrill/message_clicked_1.sql
[unsub-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.mandrill/recipient_unsubscribed_1.sql
