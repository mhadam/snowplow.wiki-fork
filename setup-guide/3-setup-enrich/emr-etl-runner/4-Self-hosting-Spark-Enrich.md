<a name="top" />

[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 3: Setting up Enrich**](Setting-up-enrich) » [Step 3.1: setting up EmrEtlRunner](Setting-up-EmrEtlRunner) » 4: Self-hosting Spark Enrich

1. [Overview](#overview)
2. [Bucket and directory setup](#bucket-setup)
3. [Uploading files](#files)
4. [Configuring EmrEtlRunner](#configure)
5. [Next steps](#next-steps)

<a name="usage-overview"/>

## 1. Overview

EmrEtlRunner runs Spark Enrich resources which are publicly hosted on Amazon S3 by Snowplow - please see the [[Hosted assets]] page for details. For most users, this will be fine. However, there are some cases where you will need to self-host the Spark Enrich process in your own Amazon S3 bucket. Two examples are:

1. You are using a custom fork of the Spark Enrich process
2. You are using a commercial version of the MaxMind GeoCity database

For self-hosting instructions read on.

<a name="bucket-setup"/>

## 2. Bucket and directory setup

First create a new S3 bucket, for example:

    s3://[mycompanyname]-snowplow-hosted-assets

You do **not** need to give any public permissions on this bucket.

Now create the following two directory structures:

    s3://[mycompanyname]-snowplow-hosted-assets/3-enrich/spark-enrich
    s3://[mycompanyname]-snowplow-hosted-assets/third-party/maxmind

That's it - you are now ready to upload your files.

<a name="files"/>

## 3. Uploading files

### 3.1 MaxMind database

If you are using the free GeoCityLite version of the MaxMind database, then download it from [[Hosted assets]] and upload to:

    s3://[mycompanyname]-snowplow-hosted-assets/third-party/maxmind/GeoLiteCity.dat

If you are using a commercial version of the MaxMind GeoCity database, then download it from your MaxMind account and upload it into this directory:

    s3://[mycompanyname]-snowplow-hosted-assets/third-party/maxmind/

**Please note:** MaxMind releases an updated version of the GeoCity database each month, so be sure to keep your version up-to-date.

### 3.2 Spark Enrich process

If you are using the standard version of Spark Enrich, then download it from [[Hosted assets]] and upload to:

    s3://[mycompanyname]-snowplow-hosted-assets/3-enrich/spark-enrich/snowplow-spark-enrich-[version].jar

If you are using a custom fork of the Spark Enrich process, then upload your assembled fatjar to:

    s3://[mycompanyname]-snowplow-hosted-assets/3-enrich/spark-enrich/snowplow-spark-enrich-[version]-[fork].jar

<a name="configure"/>

## 4. Configuring EmrEtlRunner

In your `config.yml` file, set `s3.buckets.assets` to your own bucket name:

```yaml
buckets:
  assets: s3://[mycompanyname]-snowplow-hosted-assets
```

If you are using a custom fork of the Spark Enrich process, make sure to update `enrich:versions:spark_enrich` to your own Spark Enrich version:

```yaml
enrich:
  ...
  versions:
    ...
    spark_enrich: [version]-[fork]
```

<a name="next-steps" />

## 5. Next steps

And that's it - you should now be able to run EmrEtlRunner against the custom/commercial assets hosted in your own dedicated S3 bucket.
