[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Storage**](storage-documentation) » Relational Database Loader

Relational Database Loader (RDB Loader) is an application running as AWS EMR step intended to discover shredded data on S3 and load it into relational databases, such as AWS Redshift and PostgreSQL.
RDB Loader was previously known as [StorageLoader JRuby app](StorageLoader), which it is replaced in R90 release.

The RDB Loader gets submitted to EMR via [[EmrEtlRunner]] after Spark Shred job. EmrEtlRunner passes to it [configuration file][config-file] (minus AWS credentials) and storage targets configuration.

## The RDB Loader role in ETL process

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

[config-file]: https://github.com/snowplow/snowplow/blob/master/3-enrich/emr-etl-runner/config/config.yml.sample
