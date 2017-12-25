<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2b: setup a Webhook**](Setting-up-a-webhook) > [[Urban Airship Connect webhook setup]]

## Contents

- 1 [Overview](#overview)
  - 1.1 [Compatibility](#compat)
- 2 [Setup](#setup)
  - 2.1 [UrbanAirship Connect](#setup-urbanairship)
  - 2.2 [Snowplow Redshift](#setup-redshift)
  - 2.3 [EmrEtlRunner](#setup-emr-etl-runner)

<a name="overview" />

## 1. Overview

**Technically this is not a webhook as it is integrated via S3, but we include it in this section for ease of discovery.**

This webhook integration lets you track a variety of events logged by [UrbanAirship Connect][urbanairship-website].

Available events are:

- CLOSE
- CUSTOM
- FIRST_OPEN
- IN_APP_MESSAGE_DISPLAY
- IN_APP_MESSAGE_EXPIRATION
- IN_APP_MESSAGE_RESOLUTION
- LOCATION
- OPEN
- PUSH_BODY
- REGION
- RICH_DELETE
- RICH_DELIVERY
- RICH_READ
- SEND
- TAG_CHANGE
- UNINSTALL

For the technical implementation details, see [[Urban Airship Connect webhook adapter]].

<a name="compat" />

### 1.1 Compatibility

* [Snowplow 75 Long-Legged Buzzard][snowplow-release]
* [UrbanAirship Connect S3 Integration][urbanairship-webhooks]

<a name="setup" />

## 2. Setup

Integrating UrbanAirship Connect's webhooks into Snowplow is a three-stage process:

1. Configure UrbanAirship Connect to send events to Snowplow
2. (Optional) Create the UrbanAirship Connect events tables for Amazon Redshift
3. Configure EmrEtlRunner to load UrbanAirship events

<a name="setup-urbanairship" />

## 2.1 UrbanAirship Connect

You'll need to log into your UrbanAirship account and set up integrations to a S3 bucket first.

<a name="setup-redshift" />

## 2.2 Redshift

If you are running the Snowplow batch flow with Amazon Redshift, then you should deploy the relevant event tables into your Amazon Redshift.

You can find the table definitions here:

* [com_urbanairship_CLOSE_1.sql][CLOSE-sql]
* [com_urbanairship_CUSTOM_1.sql][CUSTOM-sql]
* [com_urbanairship_FIRST_OPEN_1.sql][FIRST_OPEN-sql]
* [com_urbanairship_IN_APP_MESSAGE_DISPLAY_1.sql][IN_APP_MESSAGE_DISPLAY-sql]
* [com_urbansirship_IN_APP_MESSAGE_EXPIRATION_1.sql][IN_APP_MESSAGE_EXPIRATION-sql]
* [com_urbanairship_IN_APP_MESSAGE_RESOLUTION_1.sql][IN_APP_MESSAGE_RESOLUTION-sql]
* [com_urbanairship_LOCATION_1.sql][LOCATION-sql]
* [com_urbanairship_OPEN_1.sql][OPEN-sql]
* [com_urbanairship_PUSH_BODY_1.sql][PUSH_BODY-sql]
* [com_urbanairship_REGION_1.sql][REGION-sql]
* [com_urbanairship_RICH_DELETE_1.sql][RICH_DELETE-sql]
* [com_urbanairship_RICH_DELIVERY_1.sql][RICH_DELIVERY-sql]
* [com_urbanairship_RICH_READ_1.sql][RICH_READ-sql]
* [com_urbanairship_SEND_1.sql][SEND-sql]
* [com_urbanairship_TAG_CHANGE_1.sql][TAG_CHANGE-sql]
* [com_urbanairship_UNINSTALL_1.sql][UNINSTALL-sql]

Make sure to deploy this table into the same schema as your `events` table.

<a name="setup-emr-etl-runner" />

# 2.3 EmrEtlRunner

The minimum Spark Enrich job version is *0.19.0*. You'll need to set the version you're using to be at least this. This setting can be found in the file `config.yml`

```yaml
  versions:
    spark_enrich: 0.19.0
```

You'll also need to set your loader format. To collect Urban Airship Connect events, this should be set to `ndjson/com.urbanairship.connect/v1` as below.

```yaml
  collector_format: ndjson/com.urbanairship.connect/v1
```

Finally, you need to point EmrEtlRunner at your UrbanAirship integration S3 bucket:

```yaml
raw:
  in:
    - s3://bucket-specified-in-urban-airship
```

For a complete example, see our [sample `config.yml` template][emretlrunner-config-yml].

That's it - with these tables deployed and your configuration set up, your UrbanAirship Connect events will be ingested by running EmrEtlRunner.

[urbanairship-website]: http://urbanairship.com/
[urbanairship-webhooks]: https://docs.urbanairship.com/connect/index.html
[tracker-protocol]: https://github.com/snowplow/snowplow/wiki/snowplow-tracker-protocol#1-common-parameters-platform-and-event-independent

[urbanairship-adapter]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-common-enrich/src/main/scala/com.snowplowanalytics.snowplow.enrich/common/adapters/registry/SendgridAdapter.scala
[snowplow-release]: https://github.com/snowplow/snowplow/releases/tag/r75-long-legged-buzzard

[CLOSE-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/close_1.sql
[CUSTOM-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/custom_1.sql
[FIRST_OPEN-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/first_open_1.sql
[IN_APP_MESSAGE_DISPLAY-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/in_app_message_display_1.sql
[IN_APP_MESSAGE_EXPIRATION-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/in_app_message_expiration_1.sql
[IN_APP_MESSAGE_RESOLUTION-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/in_app_message_resolution_1.sql
[LOCATION-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/location_1.sql
[OPEN-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/open_1.sql
[PUSH_BODY-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/push_body_1.sql
[REGION-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/region_1.sql
[RICH_DELETE-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/rich_delete_1.sql
[RICH_DELIVERY-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/rich_delivery_1.sql
[RICH_READ-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/rich_read_1.sql
[SEND-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/send_1.sql
[TAG_CHANGE-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/tag_change_1.sql
[UNINSTALL-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.urbanairship.connect/uninstall_1.sql

 [emretlrunner-config-yml]: https://github.com/snowplow/snowplow/blob/master/3-enrich/emr-etl-runner/config/config.yml.sample
