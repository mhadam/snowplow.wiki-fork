<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [Step 3: Setting up Enrich](Setting-up-enrich) > [Step 3.1: setting up EmrEtlRunner](Setting-up-EmrEtlRunner) > Step 3.1.1: Installing EmrEtlRunner

1. [Assumptions](#assumptions)
2. [Dependencies](#dependencies)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Configuring enrichments](#enrichments)
6. [Next steps](#next-steps)

<a name="assumptions"/>

## 1. Assumptions

This guide assumes that you have administrator access to a Unix-based server (e.g. Ubuntu, OS X, Fedora) on which you can install EmrEtlRunner and schedule a regular cronjob.

You might wish to try out the [steps](Setting-up-EC2-instance-for-EmrEtlRunner-and-StorageLoader) showing you how an EC2 instance could be set up via [AWS CLI](https://aws.amazon.com/cli/).

_In theory EmrEtlRunner can be deployed onto a Windows-based server, using the Windows Task Scheduler instead of cron, but this has not been tested or documented._

<a name="dependencies"/>

## 2. Dependencies

### 2.1 Hardware

You will need to setup EmrEtlRunner on your own server. A number of people choose to do so on an EC2 instance (thereby keeping all of Snowplow in the Amazon Cloud). If you do so, please note that you **must not use a `t1.micro` instance**.  You should at the very least use an `m1.small` instance.

### 2.2 Software

The EmrEtlRunner jar is available for download. For more information, see the [[Hosted assets]] page.

\* If you prefer, an alternative Ruby manager such as chruby or rbenv should work fine too.

### 2.3 EC2 key

You will also need an **EC2 key pair** setup in your Amazon EMR account.

For details on how to do this, please see [Create a Key Pair](http://docs.aws.amazon.com/ElasticMapReduce/latest/DeveloperGuide/EMR_SetUp_KeyPair.html). Make sure that you setup the EC2 key pair inside the region in which you will be running your ETL jobs.

<a name="s3-buckets"/>

### 2.4 S3 locations

EmrEtlRunner processes data through three distinct states:

1. **:raw** - raw Snowplow event logs are the input to the EmrEtlRunner process
2. **:enriched** - EmrEtlRunner validates and enriches the raw event logs into enriched events
3. **:shredded** - EmrEtlRunner shreds JSONs found in enriched events ready for loading into dedicated Redshift tables

For `:raw:in`, specify the Amazon S3 path you configured for your Snowplow collector.

For all other S3 locations, you can specify paths within a single S3 bucket that you setup now. This bucket must be in the same AWS region as your `:raw:in` bucket.

Done? Right, now we can install EmrEtlRunner.

<a name="installation"/>

## 3. Installation

We host EmrEtlRunner on the distribution platform [JFrog Bintray](https://bintray.com/). You can get a copy of it as shown below.

**Note**: follow [this link](http://dl.bintray.com/snowplow/snowplow-generic/) to choose your version of the EmrEtlRunner. The distribution name follows the pattern `snowplow_emr_{{RELEASE_VERSION}}.zip`.

```sh
$ wget http://dl.bintray.com/snowplow/snowplow-generic/snowplow_emr_{{RELEASE_VERSION}}.zip
```

The archive contains both EmrEtlRunner and [StorageLoader](1-Installing-the-StorageLoader). Unzip the archive:

```sh
$ unzip snowplow_emr_{{RELEASE_VERSION}}.zip
```

The archive should contain a `snowplow-emr-etl-runner` file.

<a name="configuration"/>

## 4. Configuration

EmrEtlRunner requires a YAML format configuration file to run. There is a configuration file template available in the Snowplow GitHub repository at [`/3-enrich/emr-etl-runner/config/config.yml.sample`][config-yml]. See [[Common configuration]] more information on how to write this file.

### Storage targets

Storages for data can be configured using storage targets JSONs. Configuration file templates available in the Snowplow GitHub repository at [`/4-storage/config/targets` directory][targets-jsons]

### Iglu

You will also need an Iglu resolver configuration file. This is where we list the schema repositories to use to retrieve JSON Schemas for validation. For more information on this, see the [wiki page for Configuring shredding](6-Configuring-shredding).

<a name="enrichments" />

## 5. Configuring enrichments

If you wish to use Snowplow enrichments, see the [wiki page for configuring enrichments](5-Configuring-enrichments).

<a name="next-steps" />

## 6. Next steps

All done installing EmrEtlRunner? Then [learn how to use it][using-emretlrunner]

[git-install]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[config-yml]: https://github.com/snowplow/snowplow/blob/master/3-enrich/emr-etl-runner/config/config.yml.sample
[targets-jsons]: https://github.com/snowplow/snowplow/blob/master/4-storage/config/targets/
[using-emretlrunner]: 2-Using-EmrEtlRunner

[install-aws-cli]: http://docs.aws.amazon.com/cli/latest/userguide/installing.html
