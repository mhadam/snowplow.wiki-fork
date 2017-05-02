<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > UrbanAirship Connect webhook adapter

## Contents

- 1 [Overview](#overview)
- 2 [Implementation](#implementation)
  - 2.1 [Event source](#source)
  - 2.2 [Snowplow adapter](#adapter)
- 3 [Events](#events)
- 4 [See also](#see-also)

<a name="overview" />

## 1. Overview

**Technically this is not a webhook as it is integrated via S3, but we include it in this section for ease of discovery.**

This webhook adapter lets you track a variety of events logged by [UrbanAirship  Connect][urbanairship-website].

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

Details of the UrbanAirship webhook format as of 30th November 2015.

UrbanAirship uploads NDJSON files to a S3 bucket periodically. This is different to their advertised web API, and can be enabled using their website, under "integrations".

<a name="adapter" />

### 2.2 Snowplow adapter

Implementation: [UrbanAirship][UrbanAirship-adapter]

UrbanAirship webhook support was implemented in [Snowplow 75 Long-Legged Buzzard][snowplow-release].

<a name="events" />

## 3. Events

All resources for this webhook's events:

| **Event**                   | **JSON Schema**                                                             | **JSON Paths**                                                           | **Redshift Table**                                                                  |
|:----------------------------|:----------------------------------------------------------------------------|:-------------------------------------------------------------------------|:------------------------------------------------------------------------------------|
| CLOSE                       | [CLOSE 1-0-0][close-json-schema]                                           | [close_1.json][close-json-paths]                                        | [com_urbanairship_close_1.sql][close-sql]                                          |
| CUSTOM                      | [CUSTOM 1-0-0][custom-json-schema]                                         | [custom_1.json][custom-json-paths]                                      | [com_urbanairship_custom_1.sql][custom-sql]                                        |
| FIRST_OPEN                  | [FIRST_OPEN 1-0-0][first_open-json-schema]                                 | [first_open_1.json][first_open-json-paths]                              | [com_urbanairship_first_open_1.sql][first_open-sql]                                |
| IN_APP_MESSAGE_DISPLAY      | [IN_APP_MESSAGE_DISPLAY 1-0-0][in_app_message_display-json-schema]         | [in_app_message_display_1.json][in_app_message_display-json-paths]      | [com_urbanairship_in_app_message_display_1.sql][in_app_message_display-sql]        |
| IN_APP_MESSAGE_EXPIRATION   | [IN_APP_MESSAGE_EXPIRATION 1-0-0][in_app_message_expiration-json-schema]   | [in_app_message_expiration_1.json][in_app_message_expiration-json-paths]| [com_urbanairship_in_app_message_expiration_1.sql][in_app_message_expiration-sql]  |
| IN_APP_MESSAGE_RESOLUTION   | [IN_APP_MESSAGE_RESOLUTION 1-0-0][in_app_message_resolution-json-schema]   | [in_app_message_resolution_1.json][in_app_message_resolution-json-paths]| [com_urbanairship_in_app_message_resolution_1.sql][in_app_message_resolution-sql]  |
| LOCATION                    | [LOCATION 1-0-0][location-json-schema]                                     | [location_1.json][location-json-paths]                                  | [com_urbanairship_location_1.sql][location-sql]                                    |
| OPEN                        | [OPEN 1-0-0][open-json-schema]                                             | [open_1.json][open-json-paths]                                          | [com_urbanairship_open_1.sql][open-sql]                                            |
| PUSH_BODY                   | [PUSH_BODY 1-0-0][push_body-json-schema]                                   | [push_body_1.json][push_body-json-paths]                                | [com_urbanairship_push_body_1.sql][push_body-sql]                                  |
| REGION                      | [REGION 1-0-0][region-json-schema]                                         | [region_1.json][region-json-paths]                                      | [com_urbanairship_region_1.sql][region-sql]                                        |
| RICH_DELETE                 | [RICH_DELETE 1-0-0][rich_delete-json-schema]                               | [rich_delete_1.json][rich_delete-json-paths]                            | [com_urbanairship_rich_delete_1.sql][rich_delete-sql]                              |
| RICH_DELIVERY               | [RICH_DELIVERY 1-0-0][rich_delivery-json-schema]                           | [rich_delivery_1.json][rich_delivery-json-paths]                        | [com_urbanairship_rich_delivery_1.sql][rich_delivery-sql]                          |
| RICH_READ                   | [RICH_READ 1-0-0][rich_read-json-schema]                                   | [rich_read_1.json][rich_read-json-paths]                                | [com_urbanairship_rich_read_1.sql][rich_read-sql]                                  |
| SEND                        | [SEND 1-0-0][send-json-schema]                                             | [send_1.json][send-json-paths]                                          | [com_urbanairship_send_1.sql][send-sql]                                            |
| TAG_CHANGE                  | [TAG_CHANGE 1-0-0][tag_change-json-schema]                                 | [tag_change_1.json][tag_change-json-paths]                              | [com_urbanairship_tag_change_1.sql][tag_change-sql]                                |
| UNINSTALL                   | [UNINSTALL 1-0-0][uninstall-json-schema]                                   | [uninstall_1.json][uninstall-json-paths]                                | [com_urbanairship_uninstall_1.sql][uninstall-sql]                                  |


<a name="see-also" />

## 4. See also

[[Urban Airship Connect webhook setup]]

[urbanairship-website]: http://https://www.urbanairship.com/

[UrbanAirship-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/UrbanAirshipAdapter.scala
[snowplow-release]: http://snowplowanalytics.com/blog/2015/12/04/snowplow-r75-long-legged-buzzard-released

[close-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/close_1.json
[close-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/close/jsonschema/1-0-0
[close-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/close_1.sql

[custom-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/custom/jsonschema/1-0-0
[custom-json-paths]: https://github.com/snowplow/snowplow/tre0.9.11e/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/custom_1.json
[custom-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/custom_1.sql

[first_open-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/first_open/jsonschema/1-0-0
[first_open-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/first_open_1.json
[first_open-sql]: https://githuopen-json-pathsbpathsb.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/first_open_1.sql

[in_app_message_display-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/in_app_message_display/jsonschema/1-0-0
[in_app_message_display-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/in_app_message_display_1.json
[in_app_message_display-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/in_app_message_display_1.sql

[in_app_message_expiration-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/in_app_message_expiration/jsonschema/1-0-0
[in_app_message_expiration-json-paths]: https://github.unsubscribe-json-pathscom/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/in_app_message_expiration_1.json
[in_app_message_expiration-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/in_app_message_expiration_1.sql

[in_app_message_resolution-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/in_app_message_resolution/jsonschema/1-0-0
[in_app_message_resolution-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/in_app_message_resolution_1.json
[in_app_message_resolution-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/in_app_message_resolution_1.sql

[location-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/location/jsonschema/1-0-0
[location-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/location_1.json
[location-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/location_1.sql

[open-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/open/jsonschema/1-0-0
[open-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/open_1.json
[open-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/open_1.sql

[push_body-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/push_body/jsonschema/1-0-0
[push_body-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/push_body_1.json
[push_body-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/push_body_1.sql

[region-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/region/jsonschema/1-0-0
[region-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/region_1.json
[region-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/region_1.sql

[rich_delete-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/rich_delete/jsonschema/1-0-0
[rich_delete-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/rich_delete_1.json
[rich_delete-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/rich_delete_1.sql

[rich_delivery-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/rich_delivery/jsonschema/1-0-0
[rich_delivery-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/rich_delivery_1.json
[rich_delivery-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/rich_delivery_1.sql

[rich_read-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/rich_read/jsonschema/1-0-0
[rich_read-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/rich_read_1.json
[rich_read-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/rich_read_1.sql

[send-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/send/jsonschema/1-0-0
[send-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/send_1.json
[send-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/send_1.sql

[tag_change-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/tag_change/jsonschema/1-0-0
[tag_change-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/tag_change_1.json
[tag_change-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/tag_change_1.sql

[uninstall-json-schema]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.urbanairship.connect/uninstall/jsonschema/1-0-0
[uninstall-json-paths]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/jsonpaths/com.urbanairship.connect/uninstall_1.json
[uninstall-sql]: https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql/com.urbanairship.connect/uninstall_1.sql
