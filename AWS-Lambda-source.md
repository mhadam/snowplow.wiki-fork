<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Webhooks**](Webhooks) > AWS Lambda Source

## Contents

- 1. [Overview](#overview)
- 2. [Implementation](#implementation)
  - 2.1 [Event source](#source)
- 3. [Events](#events)
- 4. [See also](#see-also)

<a name="overview" />

## 1. Overview

This event source allows you to track S3 bucket events (file created, file deleted) on specified buckets.

<a name="implementation" />

## 2. Implementation

<a name="source" />

### 2.1 Event source

Using the AWS Lambda "push" model, the AWS Lambda source is invoked by S3 on put/delete events. A sample of the events is provided [here (under *S3 Put* and *S3 Delete*)](http://docs.aws.amazon.com/lambda/latest/dg/eventsources.html).
The JSON Schema for the above events is available on Iglu Central [here](https://github.com/snowplow/iglu-central/blob/master/schemas/com.amazon.aws.lambda/s3_notification_event/jsonschema/1-0-0).

The AWS Lambda function is written in Java 8 using gradle as a build tool. It's uploaded to AWS as a zip - and uses the `description` field of the Lambda as the collector endpoint.
The [Snowplow Java Tracker](https://github.com/snowplow/snowplow-java-tracker) is used to transmit events.

<a name="events" />

## 3. Events

All resources for the S3 source events:

| **Event**      | **JSON Schema**                                  | **JSON Paths**                                    | **Redshift Table**                                     |
|:---------------|:-------------------------------------------------|:--------------------------------------------------|:-------------------------------------------------------|
| S3 Put/Delete  | [com.amazon.aws.lambda/s3_notification_event](https://github.com/snowplow/iglu-central/blob/master/schemas/com.amazon.aws.lambda/s3_notification_event/jsonschema/1-0-0)    | [s3_notification_event_1.json](https://github.com/snowplow/snowplow/blob/3baae3f40bcfcde32106e032e5ccf5ac08a400a0/4-storage/redshift-storage/jsonpaths/com.amazon.aws.lambda/s3_notification_event_1.json)  |     [s3_notification_event_1.sql](https://github.com/snowplow/snowplow/blob/661c6a60c84b3a8af113118d162f0a29680d6904/4-storage/redshift-storage/sql/com.amazon.aws.lambda/s3_notification_event_1.sql)        |

<a name="see-also" />

## 4. See also

[[AWS Lambda Source setup]]
