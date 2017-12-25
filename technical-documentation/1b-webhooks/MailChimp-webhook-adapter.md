<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > Mailchimp webhook adapter

## Contents

- 1 [Overview](#overview)
- 2 [Implementation](#implementation)
  - 2.1 [Event source](#source)
  - 2.2 [Snowplow adapter](#adapter)
- 3 [Events](#events)
- 4 [See also](#see-also)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track a variety of events logged by [MailChimp][mailchimp-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

Details of the MailChimp webhook format as of 1 November 2014.

MailChimp sends events as a `POST` request with all information in the body, with `application/x-www-form-urlencoded` as the content type.

<a name="adapter" />

### 2.2 Snowplow adapter

Implementation: [MailChimpAdapter][mailchimp-adapter]

MailChimp webhook support was implemented in [Snowplow 0.9.11][snowplow-0.9.11].

<a name="events" />

## 3. Events

All resources for this webhook's events:

| **Event**      | **JSON Schema**                                  | **JSON Paths**                                    | **Redshift Table**                                     |
|:---------------|:-------------------------------------------------|:--------------------------------------------------|:-------------------------------------------------------|
| Subscribe               | [subscribe 1-0-1][subscribe-json-schema]               | [subscribe_1.json][subscribe-json-paths]               | [com_mailchimp_subscribe_1.sql][subscribe-sql]               |
| Unsubscribe             | [unsubscribe 1-0-1][unsubscribe-json-schema]           | [unsubscribe_1.json][unsubscribe-json-paths]           | [com_mailchimp_unsubscribe_1.sql][unsubscribe-sql]           |
| Profile Update          | [profile_update 1-0-1][profile-json-schema]            | [profile_update_1.json][profile-json-paths]            | [com_mailchimp_profile_update_1.sql][profile-sql]            |
| Email Address Change    | [email_address_change 1-0-1][email-change-json-schema] | [email_address_change_1.json][email-change-json-paths] | [com_mailchimp_email_address_change_1.sql][email-change-sql] |
| Cleaned Email           | [cleaned_email 1-0-1][email-clean-json-schema]         | [cleaned_email_1.json][email-clean-json-paths]         | [com_mailchimp_cleaned_email.sql][email-clean-sql]         |
| Campaign Sending Status | [campaign_sending_status 1-0-1][campaign-json-schema]  | [campaign_sending_status_1.json][campaign-json-paths]  | [com_mailchimp_campaign_sending_status_1.sql][campaign-sql]  |

<a name="see-also" />

## 4. See also

[[MailChimp webhook setup]]

[mailchimp-website]: http://mailchimp.com/

[mailchimp-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/MailchimpAdapter.scala
[snowplow-0.9.11]: https://github.com/snowplow/snowplow/releases/tag/0.9.11

[subscribe-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mailchimp/subscribe/jsonschema/1-0-0
[subscribe-json-paths]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailchimp/subscribe_1.json
[subscribe-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailchimp/subscribe_1.sql

[unsubscribe-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mailchimp/unsubscribe/jsonschema/1-0-0
[unsubscribe-json-paths]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailchimp/unsubscribe_1.json
[unsubscribe-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailchimp/unsubcribe_1.sql

[profile-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mailchimp/profile_update/jsonschema/1-0-0
[profile-json-paths]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailchimp/profile_update_1.json
[profile-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailchimp/profile_update_1.sql

[email-change-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mailchimp/email_address_change/jsonschema/1-0-0
[email-change-json-paths]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailchimp/email_address_change_1.json
[email-change-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailchimp/email_address_change_1.sql

[email-clean-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mailchimp/cleaned_email/jsonschema/1-0-0
[email-clean-json-paths]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailchimp/cleaned_email_1.json
[email-clean-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailchimp/cleaned_email_1.sql

[campaign-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.mailchimp/campaign_sending_status/jsonschema/1-0-0
[campaign-json-paths]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.mailchimp/campaign_sending_status_1.json
[campaign-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.mailchimp/campaign_sending_status_1.sql
