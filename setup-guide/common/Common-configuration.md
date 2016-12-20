[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow)

### Overview

This page describes the format for the YAML file which is used to configure both the EmrEtlRunner and the Storage Loader.

You can and should use the same file for both applications.

### Using environment variables

You can use environment variables rather than hardcoding strings in the configuration file. For example, load your AWS access key from an environment variable named "AWS_SNOWPLOW_SECRET_KEY":

```yaml
secret_access_key: <%= ENV['AWS_SNOWPLOW_SECRET_KEY'] %>
```

### Example configuration

```yaml
aws:
  # Credentials can be hardcoded or set in environment variables
  access_key_id: <%= ENV['AWS_SNOWPLOW_ACCESS_KEY'] %>
  secret_access_key: <%= ENV['AWS_SNOWPLOW_SECRET_KEY'] %>
  s3:
    region: ADD HERE
    buckets:
      assets: s3://snowplow-hosted-assets # DO NOT CHANGE unless you are hosting the jarfiles etc yourself in your own bucket
      jsonpath_assets: # If you have defined your own JSON Schemas, add the s3:// path to your own JSON Path files in your own bucket here
      log: ADD HERE
      raw:
        in:                  # Multiple in buckets are permitted
          - ADD HERE          # e.g. s3://my-in-bucket
          - ADD HERE
        processing: ADD HERE
        archive: ADD HERE    # e.g. s3://my-archive-bucket/in
      enriched:
        good: ADD HERE       # e.g. s3://my-out-bucket/enriched/good
        bad: ADD HERE        # e.g. s3://my-out-bucket/enriched/bad
        errors: ADD HERE     # Leave blank unless continue_on_unexpected_error: set to true below
        archive: ADD HERE    # Where to archive enriched events to, e.g. s3://my-archive-bucket/enriched
      shredded:
        good: ADD HERE       # e.g. s3://my-out-bucket/shredded/good
        bad: ADD HERE        # e.g. s3://my-out-bucket/shredded/bad
        errors: ADD HERE     # Leave blank unless continue_on_unexpected_error: set to true below
        archive: ADD HERE    # Where to archive shredded events to, e.g. s3://my-archive-bucket/shredded
  emr:
    ami_version: 4.5.0      # Don't change this
    region: ADD HERE        # Always set this
    jobflow_role: EMR_EC2_DefaultRole # Created using $ aws emr create-default-roles
    service_role: EMR_DefaultRole     # Created using $ aws emr create-default-roles
    placement: ADD HERE     # Set this if not running in VPC. Leave blank otherwise
    ec2_subnet_id: ADD HERE # Set this if running in VPC. Leave blank otherwise
    ec2_key_name: ADD HERE
    bootstrap: []           # Set this to specify custom boostrap actions. Leave empty otherwise
    software:
      hbase:                # Optional. To launch on cluster, provide version, "0.92.0", keep quotes. Leave empty otherwise.
      lingual:              # Optional. To launch on cluster, provide version, "1.1", keep quotes. Leave empty otherwise.
    # Adjust your Hadoop cluster below
    jobflow:
      master_instance_type: m1.medium
      core_instance_count: 2
      core_instance_type: m1.medium
      task_instance_count: 0 # Increase to use spot instances
      task_instance_type: m1.medium
      task_instance_bid: 0.015 # In USD. Adjust bid, or leave blank for non-spot-priced (i.e. on-demand) task instances
    bootstrap_failure_tries: 3 # Number of times to attempt the job in the event of bootstrap failures
    additional_info:        # Optional JSON string for selecting additional features
collectors:
  format: cloudfront # Or 'clj-tomcat' for the Clojure Collector, or 'thrift' for Thrift records, or 'tsv/com.amazon.aws.cloudfront/wd_access_log' for Cloudfront access logs
enrich:
  job_name: Snowplow ETL # Give your job a name
  versions:
    hadoop_enrich: 1.8.0 # Version of the Hadoop Enrichment process
    hadoop_shred: 0.10.0 # Version of the Hadoop Shredding process
    hadoop_elasticsearch: 0.1.0 # Version of the Hadoop to Elasticsearch copying process
  continue_on_unexpected_error: false # Set to 'true' (and set out_errors: above) if you don't want any exceptions thrown from ETL
  output_compression: NONE # Compression only supported with Redshift, set to NONE if you have Postgres targets. Allowed formats: NONE, GZIP
storage:
  download:
    folder: # Postgres-only config option. Where to store the downloaded files. Leave blank for Redshift
  targets:
    - name: "My Redshift database"
      type: redshift
      host: ADD HERE # The endpoint as shown in the Redshift console
      database: ADD HERE # Name of database
      port: 5439 # Default Redshift port
      table: atomic.events
      username: ADD HERE
      password: ADD HERE
      maxerror: 1 # Stop loading on first error, or increase to permit more load errors
      comprows: 200000 # Default for a 1 XL node cluster. Not used unless --include compupdate specified
      ssl_mode: disable
    - name: "My PostgreSQL database"
      type: postgres
      host: ADD HERE # Hostname of database server
      database: ADD HERE # Name of database
      port: 5432 # Default Postgres port
      table: atomic.events
      username: ADD HERE
      password: ADD HERE
      maxerror: # Not required for Postgres
      comprows: # Not required for Postgres
      ssl_mode: disable
    - name: "myelasticsearchtarget" # Name for the target - used to label the corresponding jobflow step
      type: elasticsearch # Marks the database type as Elasticsearch
      host: "ec2-43-1-854-22.compute-1.amazonaws.com" # Elasticsearch host
      database: index1 # The Elasticsearch index
      port: 9200 # Port used to connect to Elasticsearch
      table: type1 # The Elasticsearch type
      es_nodes_wan_only: false # Set to true if using Amazon Elasticsearch Service
      username: # Unnecessary for Elasticsearch
      password: # Unnecessary for Elasticsearch
      sources: # Leave blank or specify: ["s3://out/enriched/bad/run=xxx", "s3://out/shred/bad/run=yyy"]
      maxerror: # Not required for Elasticsearch
      comprows: # Not required for Elasticsearch
monitoring:
  tags: {} # Name-value pairs describing this job
  logging:
    level: DEBUG # You can optionally switch to INFO for production
  snowplow:
    method: get
    app_id: ADD HERE # e.g. snowplow
    collector: ADD HERE # e.g. d3rkrsqld9gmqf.cloudfront.net

```

### aws

#### Credentials

The `access_key_id` and `secret_access_key` variables should be self-explanatory - enter your AWS access key and secret here.

#### s3

The `region` variable should hold the AWS region in which your four data buckets (In Bucket, Processing Bucket etc) are located, e.g. "us-east-1" or "eu-west-1". Please note that Redshift can only load data from S3 buckets located in the same region as the Redshift instance, and Amazon has not to date launched Redshift in *every* region. So make sure that if you're using Redshift, the bucket specified here is in a region that supports Redshift.

Within the `s3` section, the `buckets` variables are as follows:

* `assets:` holds the ETL job's static assets (HiveQL script plus Hive deserializer). You can leave this as-is (pointing to Snowplow   Analytics' [own public bucket containing these assets](Hosted-assets)) or replace this with your own private bucket containing the assets
* `log:` is the bucket in which Amazon EMR will record processing information for this job run, including logging any errors  
* `raw:` is where you specify the paths through which your raw Snowplow events will flow. `in` is an array of one or more buckets containing raw events. For `processing:`, **always include a sub-folder on this variable (see below for why)**. `archive:` is where your raw Snowplow events will be moved after they have been successfully processed by Elastic MapReduce
* `enriched:` is where you specify the paths through which your enriched Snowplow events will flow.
* `shredded:` is where you specify the paths through which your shredded types will flow

For `good:`, **always include a sub-folder on this variable (see below for why)**. If you are loading data into Redshift, the `good:` specified here **must** be located in a region where Amazon has launched Redshift, because Redshift can only bulk load data from S3 that is located in the same region as the Redshift instance, and Redshift has not, to-date, been launched across all Amazon regions

Each of the bucket variables must start with an S3 protocol - either `s3://` or `s3n://`. Each variable can include a sub-folder within the bucket as required, and a trailing slash is optional.

The `bad:` entries will store any raw Snowplow log lines which did not pass the enrichment or JSON validation, along with their validation errors. The `errors:` entries will contain any raw Snowplow log lines which caused an unexpected error, but only if you set continue_on_unexpected_error to true (see below).

**Important 1:** there is a bug in Hive on Amazon EMR where Hive dies if you attempt to read or write data to the root of an S3 bucket. **Therefore always specify a sub-folder (e.g. `/events/`) for the `raw:processing`, `enriched:good` and `shredded:good` locations.**

**Important 2:** do not put your `raw:processing` inside your `raw:in` bucket, or your `enriched:good` inside your `raw:processing`, or you will create circular references which EmrEtlRunner cannot resolve when moving files.

**Important 3:** if you are using the **Clojure collector**, the path to your `raw:in` path will be of the format: 

	s3://elasticbeanstalk-{{REGION NAME}}-{{UUID}}/resources/environments/logs/publish/{{SECURITY GROUP IDENTIFIER}}

Replace all of these `{{x}}` variables with the appropriate ones for your environment (which you should have written down in the [Enable logging to S3](Enable-logging-to-S3) stage of the Clojure Collector setup).

Also - Clojure Collector usees should be sure not include an `{{INSTANCE IDENTIFIER}}` at the end of your path. This is because your Clojure Collector may end up logging into multiple `{{INSTANCE IDENTIFIER}}` folders. (If e.g. Elastic Beanstalk spins up more instances to run the Clojure collector, to cope with a spike in traffic.) By specifying your In Bucket only to the level of the Security Group identifier, you make sure that Snowplow can process all logs from all instances. (Because the EmrEtlRunner will process all logs in all subfolders.)

**Example bucket settings**

Here is an example configuration: 

```yaml
buckets:
  assets: s3://snowplow-hosted-assets
  log: s3n://my-snowplow-etl/logs/
  raw:
    in: [s3n://my-snowplow-logs/]
    processing: s3n://my-snowplow-etl/processing/
    archive: s3://my-archive-bucket/raw
  enriched:
    good: s3://my-data-bucket/enriched/good
    bad: s3://my-data-bucket/enriched/bad
    errors: s3://my-data-bucket/enriched/errors
    archive: s3://my-data-bucket/enriched/archive
  shredded:
    good: s3://my-data-bucket/shredded/good
    bad: s3://my-data-bucket/shredded/bad
    errors: s3://my-data-bucket/shredded/errors
```

Please note that all buckets must exist prior to running EmrEtlRunner; trailing slashes are optional.

#### emr

The EmrEtlRunner makes use of Amazon Elastic Mapreduce (EMR) to process the raw log files and output the cleaned, enriched Snowplow events table.

This section of the config file is where we configure the operation of EMR. The variables with defaults can typically be left as-is, but you will need to set:

1. `region`, which is the Amazon EC2 region in which the job should run, e.g. "us-east-1" or "eu-west-1"
2. `ec2_key_name`, which is the name of the Amazon EC2 key that you
   set up in the [[Dependencies|1-Installing-EmrEtlRunner#dependencies]] above

Make sure that the EC2 key you specify belongs in the region you specify, or else EMR won't be able to find the key. **It's strongly recommended that you choose the same Amazon region as your S3 buckets are located in.**

Since 6th April 2015, all new Elastic MapReduce users have been required to use IAM roles with EMR. You can leave the two `..._role` fields as they are, however you must first create these default EMR roles using the AWS Command Line Interface ([installation-instructions] [install-aws-cli]), like so:

    $ aws emr create-default-roles

Additionally, fill in **one** of these two:

*  `placement`, which is the Amazon EC2 region **and** availability zone
   in which the job should run, e.g. "us-east-1a" or "eu-west-1b"
*  `ec2_subnet_id`, which is the ID of the Amazon EC2 subnet you want
   to run the job in

You only need to set one of these (they are mutually exclusive settings), but you must set one.

The `software:` section lets you start up Lingual and/or HBase when you start up your Elastic MapReduce cluster. This is the configuration to start up both, specifying the versions to start:

```yaml
software:
  hbase: "0.92.0"
  lingual: "1.1"
```

### collectors

The `format` field describes the format of the EmrEtlRunner's input. The options are:

* "cloudfront" for the Cloudfront Collector
* "clj-tomcat" for the Clojure Collector
* "thrift" for Thrift raw events
* "tsv/com.amazon.aws.cloudfront/wd_access_log" for Cloudfront access logs

See the [[EmrEtlRunner Input Formats]] page.

### enrich

* `job_name`: the name to give our ETL job. This makes it easier to identify your ETL job in the Elastic MapReduce console
* `hadoop_enrich`: version of the Scala Hadoop Enrich jar
* `hadoop_shred`: version of the Scala Hadoop Shred jar
* `continue_on_unexpected_error`: continue processing even on unexpected row-level errors, e.g. an input file not matching the expected CloudFront format. Off ("false") by default

### storage

#### download

This is where we configure the StorageLoader download operation, which
downloads the Snowplow event files from Amazon S3 to your local server, 
ready for loading into your database.

This setting is needed for Postgres, but not if you are only loading into Redshift
- you can safely leave it blank.

You will need to set the `folder` variable to a local directory path -
please make sure that:

* this path exists, 
* is writable by StorageLoader
* it is empty
* **PostgreSQL's own `postgres` user must to be able to read every parent directory of the directory specified. This is necessary to ensure that PostgreSQL can read the data in the directory, when it comes to ingest it**

#### targets

In this section we configure exactly what database(s) StorageLoader should
load our Snowplow events into. At the moment, StorageLoader supports
only two types of load target, Redshift and Postgres, which require slightly different configurations.

Additionally, you can configure Elasticsearch targets in this section. These targets are ignored by StorageLoader, but the EmrEtlRunner will load bad rows (rows which fail the enrichment process) into these targets so you can more easily inspect their associated error information and determine what went wrong.

You can load multiple storage targets.

##### Redshift and Postgres

To take each variable in turn:

1. `name`, enter a descriptive name for this Snowplow storage target
2. `type`, what type of database are we loading into? Currently the
   only supported formats are "postgres" and "redshift"
3. `host`, the host (endpoint in Redshift parlance) of the databse to
   load.
4. `database`, the name of the database to load
5. `port`, the port of the database to load. 5439 is the default Redshift
   port; 5432 is the default Postgres port
6. `table`, the name of the database table which will store your
   Snowplow events. Must have been setup previously  
7. `username`, the database user to load your Snowplow events with.
   You can leave this blank to default to the user running the script
8. `password`, the password for the database user. Leave blank if there
   is no password
9. `maxerror`, a Redshift-specific setting governing how many load errors
   should be permitted before failing the overall load. See the
   [Redshift `COPY` documentation] [redshift-copy] for more details
10. `comprows`, a Redshift-specific setting defining number of rows to be used as the sample size for compression analysis
11. `ssl_mode`, determines how to handle encryption for client connections and server certificate verification.      The the following `sslmode` values are supported:
 - `disable`: SSL is disabled and the connection is not encrypted.
 - `require`: SSL is required.
 - `verify-ca`: SSL must be used and the server certificate must be verified.
 - `verify-full`: SSL must be used. The server certificate must be verified and the server hostname must match the hostname attribute on the certificate.

Note: The difference between `verify-ca` and `verify-full` depends on the policy of the root CA. If a public CA is used, `verify-ca` allows connections to a server that somebody else may have registered with the CA to succeed. In this case, `verify-full` should always be used. If a local CA is used, or even a self-signed certificate, using `verify-ca` often provides enough protection.

##### Elasticsearch

To take each variable in turn:

1. `name`: a descriptive name for this Snowplow storage target
2. `type`: should be "elasticsearch" to signal that this is an Elasticsearch target
3. `port`: The port to load. Normally 9200, should be 80 for Amazon Elasticsearch Service. 
4. `database`: The Elasticsearch index to load
5. `table`: The Elasticsearch type to load
6: `sources`: If this field is left blank, then after the enrich and shred steps of the current job, all bad rows generated by those two steps will be loaded into Elasticsearch. Otherwise, you can provide an array of buckets (such as  `["s3://out/enriched/bad/run=2015-11-04-02-52-59", "s3://out/shred/bad/run=2015-11-04-02-52-59"]` ). All bad row files in those buckets will be loaded into Elasticsearch.
7. es_nodes_wan_only: if this is set to true, the EMR job will disable node discovery. This option is necessary when using Amazon Elasticsearch Service.

For information on setting up Elasticsearch itself, see [[Setting up Amazon Elasticsearch Service]].

### monitoring

This section deals with metadata around the EmrEtlRunner and StorageLoader.

* `tags`: a dictionary of name-value pairs describing the job
* `logging`: how verbose/chatty the log output from EmrEtlRunner should be.

#### snowplow

The `snowplow` section allows the ETL apps to send Snowplow events describing their own progress. To disable this internal tracking, remove the "snowplow" field from the configuration.

* `method`: "get" or "post"
* `app_id`: ID for the pipeline
* `collector`: Endpoint to which events should be sent

[git-install]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[config-yml]: https://github.com/snowplow/snowplow/blob/master/3-enrich/emr-etl-runner/config/config.yml.sample
[using-emretlrunner]: 2-Using-EmrEtlRunner
[install-aws-cli]: http://docs.aws.amazon.com/cli/latest/userguide/installing.html
[redshift-copy]: http://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html