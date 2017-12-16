<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > [[Olark webhook adapter]]

## Contents

- 1 [Overview](#overview)
- 2 [Implementation](#implementation)
  - 2.1 [Event source](#source)
  - 2.2 [Snowplow adapter](#adapter)
- 3 [Events](#events)
- 4 [See also](#see-also)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track a variety of events logged by [olark][olark-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

Details of the olark webhook format as of 15.12.2017.

Olark sends multiple events together as a `POST` request with all information in the body, with `application/x-www-form-urlencoded` as the content type.

<a name="adapter" />

### 2.2 Snowplow adapter

Implementation: [OlarkAdapter][olark-adapter]

Olark webhook support was implemented in [R97 Knossos][r97].

<a name="events" />

## 3. Events

All resources for this webhook's events:

| **Event**      | **JSON Schema**                                  | **JSON Paths**                                    | **Redshift Table**                                     |
|:---------------|:-------------------------------------------------|:--------------------------------------------------|:-------------------------------------------------------|
|Offline Messages       | |[com_olark_offline_message_1 1-0-0][com_olark_offline_message_1-schema]               | [com_olark_offline_message_1.json][com_olark_offline_message_1-json]               | [com_olark_offline_message_1.sql] [com_olark_offline_message_1-sql]               |
|Transcript             | |[com_olark_transcript_1 1-0-0][com_olark_transcript_1-schema]                         | [com_olark_transcript_1.json][com_olark_transcript_1-json]                         | [com_olark_transcript_1.sql] [com_olark_transcript_1-sql]                         |


<a name="see-also" />

## 4. See also

[[olark webhook setup]]

[olark-website]: https://www.olark.com/
[olark-webhooks]: https://www.olark.com/help/webhooks
[r97]: https://github.com/snowplow/snowplow/releases/tag/r97-knossos
[olark-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/OlarkAdapter.scala

[com_olark_offline_message_1-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.olark/offline_message/jsonschema/1-0-0
[com_olark_transcript_1-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.olark/transcript/jsonschema/1-0-0

[com_olark_offline_message_1-json]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.olark/offline_message_1.json
[com_olark_transcript_1-json]: https://github.com/snowplow/iglu-central/blob/master/jsonpaths/com.olark/transcript_1.json

[com_olark_offline_message_1-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.olark/offline_message_1.sql
[com_olark_transcript_1-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.olark/transcript_1.sql
