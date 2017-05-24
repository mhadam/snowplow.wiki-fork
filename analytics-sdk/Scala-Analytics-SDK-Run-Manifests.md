<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Snowplow Analytics SDK**](Snowplow-Analytics-SDK) » Scala Analytics SDK Run Manifests

## Overview

The [Snowplow Analytics SDK for Scala](https://github.com/snowplow/snowplow-scala-analytics-sdk) provides you an API to work with run manifests.
Run manifests is simple way to mark chunk (particular run) of enriched data as being processed, by for example Apache Spark data-modeling job.

## Usage

Run manifests functionality resides in new `com.snowplowanalytics.snowplow.analytics.scalasdk.RunManifests` module.

Main class is `RunManifests`, that proides access to DynamoDB table via `contains` and `add`, as well as `create` method to initialize table with appropriate settings.
Other commonly-used function is `list_runids` that is gives S3 client and path to folder such as `enriched.archive` or `shredded.archive` from `config.yml` lists all 
folders that match Snowplow run id format (`run-YYYY-mm-DD-hh-MM-SS`).
Using `listRunids` and `RunManifests` you can list job runs and safely process them one by one without risk of reprocessing.

## Example

Here's a short usage example:

```scala
import com.amazonaws.services.s3.AmazonS3ClientBuilder
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBAsyncClientBuilder
import com.amazonaws.auth.DefaultAWSCredentialsProviderChain
import com.snowplowanalytics.snowplow.analytics.scalasdk.RunManifests

val DynamodbRunManifestsTable = "snowplow-run-manifests"
val EnrichedEventsArchive = "s3://acme-snowplow-data/storage/enriched-archive/"

val s3Client = AmazonS3ClientBuilder.standard()
  .withCredentials(DefaultAWSCredentialsProviderChain.getInstance)
  .build()

val dynamodbClient = AmazonDynamoDBAsyncClientBuilder.standard()
  .withCredentials(DefaultAWSCredentialsProviderChain.getInstance)
  .build()

val runManifestsTable = RunManifests(dynamodbClient, DynamodbRunManifestsTable)
runManifestsTable.create()

val unprocessed = RunManifests.listRunIds(s3Client, EnrichedEventsArchive)
  .filterNot(runManifestsTable.contains)

unprocessed.foreach { runId =>
  process(runId)
  runManifestsTable.add(runId)
}
```

In above example, we create two AWS service clients for S3 (to list job runs) and for DynamoDB (to access manifests).
These cliens are provided via AWS Java SDK and can be initialized with static credentials or with system-provided credentials.

Then we list all run ids in particular S3 path and process (by user-provided `process` function) only those that were not processed already.
Note that `runId` is simple string with S3 key of particular job run.

`RunManifests` class is a simple API wrapper to DynamoDB, using which you can:

* `create` DynamoDB table for manifests, 
* `add` run to table 
* check if table `contains` run id

[Back to top](#top)  
[Back to Scala Analytics SDK contents][contents]

[contents]: Scala-Analytics-SDK
