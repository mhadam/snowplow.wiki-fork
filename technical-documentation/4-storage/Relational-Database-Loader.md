[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Storage**](storage-documentation) » Relational Database Loader

Relational Database Loader (RDB Loader) is an application running as AWS EMR step intended to discover shredded data on S3 and load it into relational databases, such as AWS Redshift and PostgreSQL.
RDB Loader was previously known as [StorageLoader JRuby app](StorageLoader), which it is replaced in R90 release.

The RDB Loader gets submitted to EMR via [[EmrEtlRunner]] after Spark Shred job. EmrEtlRunner passes to it [configuration file][config-file] (minus AWS credentials) and storage targets configuration.

- 1 [The RDB Loader role in ETL process](#role)
- 2 [Setup](#setup)
- 3 [Usage](#usage)
- 4 [Load process](#load-process)

<a name="role" />

## 1. The RDB Loader role in ETL process

The enriched files contain the tab-separated values contributing to `atomic.events` and custom tables. The [shredding process](Shredding)

1. reads Snowplow enriched events from enriched good files (produced and temporary stored in HDFS as a result of [enrichment process](The-enrichment-process));
2. extracts any unstructured (self-describing) event JSONs and contexts JSONs found;
3. validates that these JSONs conform to the corresponding schemas located in [Iglu registry](Iglu-registry);
4. adds metadata to these JSONs to track their origins;
5. writes these JSONs out to nested folders on S3 dependent on their schema.

As a result the **enriched good** file is "shredded" into a few **shredded good** files (provided the event file contained data from at least one of the following: [custom self-describing events](Custom-events#unstructured-event), [contexts](Contexts-overview), [configurable enrichments](configurable-enrichments)):

1. a TSV formatted file containing the data for `atomic.events` table;
2. possibly one or more JSON files related to custom user specific (self-describing) events extracted from `unstruct_event` field of the enriched good file;
3. possibly one or more JSON files related to custom contexts extracted from `contexts` filed of the enriched good file;
4. possibly one or more JSON files related to configurable enrichments (if any was enabled) extracted from `derived_contexts` field of the enriched good file.

Those files end up in S3 and are used to load the data into Redshift tables dedicated to each of the above files under the RDB Loader orchestration.

The whole process could be depicted with the following dataflow diagram.

[[/technical-documentation/images/storage-loader-dataflow.png]]

<a name="setup" />

## 2. Setup

RDB Loader does not have access to AWS credentials (they can be erased from `config.yml`).
To perform `COPY FROM s3` statement, Redshift needs a read access to `shredded.good` S3 bucket.
This access can be retrieved through [AWS IAM role][redshift-iam].

To create an IAM Role you need to go to AWS Console -> IAM -> Roles -> Create new role.
Then you need chose Amazon Redshift -> `AmazonS3ReadOnlyAccess`, choose a role name, for example `RedshiftLoadRole`. Once created, copy the Role ARN as you will need it in the next section.
Now you need to attach new role to running Redshift cluster. Go to AWS Console -> Redshift -> Clusters -> Manage IAM Roles -> Attach just created role.

You need to add just created role ARN as `roleArn` in [Redshift Storage Traget][target-config] configuration JSON. 
Note: this should be full ARN URI, looking like `arn:aws:iam::719197435995:role/RedshiftLoadRole`.

<a name="usage" />

## 3. Usage

RDB Loader need to be used as an EMR step submitted by [[EmrEtlRunner]], which exactly knows what arguments need to be passed. 
In classic batch pipeline there's no need to know about any RDB Loader internals, everything should be handled by EmrEtlRunner.
However in some cases it is possible to use it from local machine.

RDB Loader accepts following options:

* `--config <config.yml>` accepts full **base64-encoded** Snowplow `config.yml` withou AWS credentials. AWS Credentials Chain used instead to get access. Required
* `--target <target.json>` accepts full **base64-encoded** [Snowplow Storage Target][target-config] JSON configuration. RDB Loader can load only one target at time. Required
* `--resolver <resolver.json>` accepts full **base64-encoded** Iglu Resolver JSON configuration. Used to validate Storage Target JSON config. Required
* `--logkey <s3key>` accepts path to S3 key, where end output will be dumbed and later can be grabbed by EmrEtlRunner. Optional since 0.14.0
* `--include <step>` list of optional steps to include
* `--skip <step>` list of enabled by-default steps to skip
* `--folder <s3folder>` accepts path to particular run folder on S3 to load
* `--dry-run` makes RDB Loader to only print load SQL statements and not actually perform any DB IO

Available steps to skip:

* `analyze` - applicaple only to Redshift. Do not perform `ANALYZE` after load
* `consistency_check` - don't wait until S3 becomes consistent. Can save up to 1 hour on loads with big shredded types cardinality, but increases possibility to fail to actual load.

Available steps to include: 

* `vacuum` - applicaple only to Redshift. Perform `VACUUM` after load

To manually pass `--config`, `--target` and `--resolver` it is required to encode content of these files with Base64.

[[EmrEtlRunner]] (as of R95) cannot pass `--dry-run`, `--folder` and `--skip consistency_check`, these options are exclusively for manual use.

<a name="load-process" />

## 4. Load process

RDB Loader internally run has four steps:

1. Configuration parsing. Parse all arguments. If this step did not succeed - it won't dump data to `logkey` path and EmrEtlRunner won't be able to print it to stdout. Debug output however can be retrieved from EMR `stdout`.
2. Discovering. Find run folders in `shredded.good`, discover shredded types and atomic events there. Only non-empty files are taken into account. If no files in `atomic-events` found - RDB Loader will exit with error.
3. Loading. Connect to database and perform `COPY`, `ANALYZE` and `VACUUM` statements for atomic and shredded events. Insert entity into `manifest` table.
4. Log file dumping and monitoring.


[config-file]: https://github.com/snowplow/snowplow/blob/master/3-enrich/emr-etl-runner/config/config.yml.sample
[target-config]: https://github.com/snowplow/snowplow/wiki/Configuring-storage-targets
[redshift-iam]: http://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-create-an-iam-role.html
