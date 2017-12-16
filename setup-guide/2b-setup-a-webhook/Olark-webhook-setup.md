<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2b: setup a Webhook**](Setting-up-a-webhook) > [[Olark webhook setup]]

## Contents

- 1 [Overview](#overview)
  - 1.1 [Compatibility](#compat)
- 2 [Setup](#setup)
  - 2.1 [Olark](#setup-Olark)
  - 2.2 [Snowplow Redshift](#setup-redshift)

<a name="overview" />

## 1. Overview

This webhook integration lets you receive [Olark][olark-website] events.

Supported events are:

- Transcripts
- Offline messages

For the technical implementation, see [[Olark webhook adapter]].

<a name="compat" />

### 1.1 Compatibility

* [R97 Knossos][r97]+ (`POST`-capable collectors only)
* [Olark webhook API][olark-webhooks]

<a name="setup" />

## 2. Setup

Integrating Olark's webhooks into Snowplow is a two-stage process:

1. Configure Olark to send events to Snowplow
2. (Optional) Create the Olark events tables into Amazon Redshift

<a name="setup-Olark" />

## 2.1 Olark

First login to Olark and click on **Integrations**:

[[/setup-guide/images/webhooks/olark/olark-1.png]]

In integrations search start typing `webhooks` (1) until the Webhooks integration is visible, then click **Edit** (2):

[[/setup-guide/images/webhooks/olark/olark-2.png]]

On the webhooks integration page select the **URL to post to** field (1)

* For the this field you will need to provide the URL to your Snowplow Collector.  We use a special path to tell Snowplow that these events are generated by Olark:

```
http://<collector host>/com.olark/v1
```

* Then select the **Send all transcripts automatically** and/or **Send offline messages** according to your needs (2). As of the time of writing (15.12.2017) no other events are directly supported so **do not tick Send all events**.

If you want, you can also manually override the event's `platform` parameter like so:

```
http://<collector host>/com.olark/v1?p=<platform code>
```

Supported platform codes can again be found in the [Snowplow Tracker Protocol][tracker-protocol]; if not set, then the value for `platform` will default to `srv` for a server-side application.

Once you click the **Save** button you are ready to receive events about your client chat interactions from olark.

<a name="setup-redshift" />

## 2.2 Redshift

If you are running the Snowplow batch flow with Amazon Redshift, then you should deploy the relevant event tables into your Amazon Redshift.

You can find the table definitions here:

* [com_olark_offline_message_1.sql][com_olark_offline_message_1-sql]
* [com_olark_transcript_1.sql][com_olark_transcript_1-sql]

Make sure to deploy this table into the same schema as your `events` table.

That's it - with this table deployed, your Mailgun events should automatically flow through into Redshift.

[olark-website]: https://www.olark.com/
[Olark-webhooks]: https://www.olark.com/help/webhooks
[r97]: https://github.com/snowplow/snowplow/releases/tag/r97-knossos

[tracker-protocol]: https://github.com/snowplow/snowplow/wiki/snowplow-tracker-protocol#1-common-parameters-platform-and-event-independent
[com_olark_offline_message_1-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.olark/offline_message_1.sql
[com_olark_transcript_1-sql]: https://github.com/snowplow/iglu-central/blob/master/sql/com.olark/transcript_1.sql