<a name="top" />

[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » Step 4: Setup alternative data stores

[[https://d3i6fms1cm1j0i.cloudfront.net/github-wiki/images/snowplow-architecture-4-storage.png]]

Snowplow supports storing your data into four different data stores:

| **Storage**                          | **Description**                                                                                               | **Status**       |
|:-------------------------------------|:--------------------------------------------------------------------------------------------------------------|:-----------------|
| S3 (EMR, [Kinesis][kinesis])         | Data is stored in the S3 file system where it can be analysed using [EMR][emr] (e.g. Hive, Spark)             | Production-ready |
| [Redshift][setup-redshift]           | A columnar database offered as a service on EMR. Optimized for performing OLAP analysis. Scales to Petabytes  | Production-ready |
| [PostgreSQL][setup-postgres]         | A popular, open source, RDBMS database                                                                        | Production-ready |
| [Elasticsearch][setup-elasticsearch] | A search server for JSON documents                                                                            | Production-ready |
| [Amazon DynamoDB][setup-dynamodb]    | Fast NoSQL database used as auxilliary storage for [de-duplication][deduplication] process                    | Production-ready |

By [setting up the EmrEtlRunner](setting-up-EmrEtlRunner) (in the previous step), you are already successfully loading your Snowplow event data into S3 where it is accessible to EMR for analysis.

If you wish to analyse your data using a wider range of tools (e.g. BI tools like [Looker][looker], [ChartIO][chartio] or [Tableau][tableau], or statistical tools like [R][r]), you will want to load your data into a database like Amazon [Redshift][setup-redshift] or [PostgreSQL][setup-postgres] to enable use of these tools.

The [RDB Loader](Relational-Database-Loader) is an EMR step to make it simple to keep an updated copy of your data in Redshift. To setup Snowplow to automatically populate a PostgreSQL and/or Redshift database with Snowplow data, you need to first:

1. [Create a database and table for Snowplow data in Redshift][setup-redshift] and/or
2. [Create a database and table for Snowplow data in PostgreSQL][setup-postgres]

Then, *afterwards*, you will need to [set up the EmrEtlRunner to regularly transfer Snowplow data into your new store(s)][storage-loader-setup]

Note that instructions on setting up both Redshift and PostreSQL on EC2 are included in this setup guide and referenced from the links above.

All done? Then get started with [data modeling][modeling] or [start analysing your event-level data][analyse].

**Note**: We recommend running all Snowplow AWS operations through an IAM user with the bare minimum permissions required to run Snowplow. Please see our [IAM user setup page](IAM-setup) for more information on doing this.

[emr]: http://aws.amazon.com/elasticmapreduce/
[kinesis]: snowplow-s3-loader-setup
[redshift]: http://aws.amazon.com/redshift/
[chartio]: http://chartio.com/
[setup-redshift]: setting-up-redshift
[storage-loader-setup]: 1-Installing-the-StorageLoader
[tableau]: http://www.tableausoftware.com/
[analyse]: Setting-up-Snowplow#step6
[modeling]: Setting-up-Snowplow#step5
[r]: http://www.r-project.org/
[looker]: http://www.looker.com/
[setup-postgres]: Setting-up-PostgreSQL
[setup-elasticsearch]: elasticsearch-loader-setup
[setup-dynamodb]: Setting-up-Amazon-DynamoDB
[deduplication]: Relational-Database-Shredder#crossbatch-deduplication
