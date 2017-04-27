<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [Step 3: Setting up Enrich](Setting-up-enrich) > [**Step 3.1: setting up EmrEtlRunner**](Setting-up-EmrEtlRunner) > [**1: Installing EmrEtlRunner**](1-Installing-EmrEtlRunner) > [EmrEtlRunner input formats](EmrEtlRunner-Input-Formats)

<a name="overview"/>

## 1. Supported input formats

Supported input formats for the EmrEtlRunner are as follows:

<a name="cloudfront"/>

### 1.1 `cloudfront`

Use this when you are running the CloudFront Collector.

Documentation:

* [[Setting up the Cloudfront collector]]

<a name="clj-tomcat"/>

### 1.2 `clj-tomcat`

Use this when you are running the Clojure Collector on Elastic Beanstalk.

Documentation:

* [[Setting up the Clojure collector]]

<a name="thrift"/>

### 1.3 `thrift`

Use this when you are using the Scala Stream Collector plus Kinesis LZO S3 Sink.

Documentation:

* [[Setting up the Scala Stream Collector]]
* [[Kinesis LZO S3 Sink Setup]]

<a name="tsv/com.amazon.aws.cloudfront/wd_access_log"/>

### 1.4 `tsv/com.amazon.aws.cloudfront/wd_access_log`

Use this when you are analyzing Amazon CloudFront access logs (web distribution format only).

If you use CloudFront as your CDN for web content, you can use Snowplow to process your CloudFront access logs. Snowplow will enrich these logs with the user-agent, page URI fragments and geo-location as standard.

To process CloudFront access logs, first create a new EmrEtlRunner `config.yml`:

1. Set your `:raw:in:` bucket to where your logs are written
2. Set your `:etl:collector_format:` to `tsv/com.amazon.aws.cloudfront/wd_access_log`
3. Provide new bucket paths and a new job name, to prevent this job from clashing with your existing Snowplow job(s)

If you are running the Snowplow batch (Hadoop) flow with Amazon Redshift, then you should deploy the relevent event table into your Amazon Redshift database. You can find the table definition here:

* [com_amazon_aws_cloudfront_wd_access_log_1.sql][access-log-sql]

You can either load these events using your existing `atomic.events` table, or if you prefer load into an all-new database or schema. If you load into your existing `atomic.events` table, make sure to schedule these loads so that they don't clash with your existing loads.

<a name="ndjson/urbanairship.connect/v1"/>

### 1.5 `ndjson/urbanairship.connect/v1`

Use this when you are working with Urban Airship Connect. For more details see [[Urban Airship Connect webhook setup]].

[access-log-sql]: https://github.com/snowplow/snowplow/blob/master/4-storage/redshift-storage/sql/com.amazon.aws.cloudfront/wd_access_log_1.sql
