<a name="top" />

[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 4: setting up alternative data stores**](Setting-up-alternative-data-stores) > Configuring storage targets

Snowplow offers the option to configure certain storage targets. This is done using configuration JSONs. 
When running EmrEtlRunner, the `--targets` argument should be populated with the filepath of a directory containing your configuration JSONs. 
Each storage target JSON file can have arbitrary name, but must conform it's JSON Schema.

Some targets are handled by EmrEtlRunner (duplicate tracking, failure tracking) and some by RDB Loader (enriched data).

Here's a list of currently supported targets, grouped by purpose:

* Enriched data
  * [Redshift](#redshift)
  * [PostgreSQL](#postgres)
  * [Snowflake](#snowflake)
* Failures
  * [ElasticSearch](#elasticsearch)
* Duplicate tracking
  * [AWS DynamoDB](#dynamodb)


<a name="redshift" />

### Redshift

Schema: [iglu:com.snowplowanalytics.snowplow.storage/redshift_config/jsonschema/2-1-0][redshift-schema]

1. `name`, a descriptive name for this Snowplow storage target
2. `host`, the host (endpoint in Redshift parlance) of the databse to load.
3. `database`, the name of the database to load
4. `port`, the port of the database to load. 5439 is the default Redshift port
5. `schema`, the name of the database schema which will store your Snowplow tables
6. `username`, the database user to load your Snowplow events with. You can leave this blank to default to the user running the script
7. `password`, the password for the database user. Either plain-text password or [`ec2ParameterStore` object][secure-redshift-password]
8. `maxError`, a Redshift-specific setting governing how many load errors should be permitted before failing the overall load. See the [Redshift `COPY` documentation][redshift-copy] for more details
9. `compRows`, a Redshift-specific setting defining number of rows to be used as the sample size for compression analysis. Should be between 1000 and 1000000000
10. `purpose`: common for all targets. Redshift supports only `ENRICHED_EVENTS`
11. `sslMode`, determines how to handle encryption for client connections and server certificate verification. The the following `sslMode` values are supported:
 - `DISABLE`: SSL is disabled and the connection is not encrypted.
 - `REQUIRE`: SSL is required.
 - `VERIFY_CA`: SSL must be used and the server certificate must be verified.
 - `VERIFY_FULL`: SSL must be used. The server certificate must be verified and the server hostname must match the hostname attribute on the certificate.
12. `roleArn`: AWS Role ARN allowing Redshift to read data from S3
13. `sshTunnel`: optional bastion host configuration

Note: The difference between `VERIFY_CA` and `VERIFY_FULL` depends on the policy of the root CA. If a public CA is used, `VERIFY_CA` allows connections to a server that somebody else may have registered with the CA to succeed. In this case, `verify-full` should always be used. If a local CA is used, or even a self-signed certificate, using `VERIFY_CA` often provides enough protection.


<a name="postgres" />

### Postgres

Schema: [iglu:com.snowplowanalytics.snowplow.storage/postgresql_config/jsonschema/1-0-1][postgresql-schema]

1. `name`, enter a descriptive name for this Snowplow storage target
2. `host`, the host (endpoint in Redshift parlance) of the databse to
   load.
3. `database`, the name of the database to load
4. `port`, the port of the database to load. 5439 is the default Redshift
   port; 5432 is the default Postgres port
5. `schema`, the name of the database schema which will store your Snowplow tables
6. `username`, the database user to load your Snowplow events with.
   You can leave this blank to default to the user running the script
7. `password`, the password for the database user. Leave blank if there
   is no password
8. `sslSode`, determines how to handle encryption for client connections and server certificate verification. The the following `sslMode` values are supported:
 - `DISABLE`: SSL is disabled and the connection is not encrypted.
 - `REQUIRE`: SSL is required.
 - `VERIFY_CA`: SSL must be used and the server certificate must be verified.
 - `VERIFY_FULL`: SSL must be used. The server certificate must be verified and the server hostname must match the hostname attribute on the certificate.
10. `purpose`: common for all targets. PostgreSQL supports only `ENRICHED_EVENTS`
11. `id`: optional machine-readable config id
12. `sshTunnel`: optional bastion host configuration

<a name="snowflake" />

### Snowflake

Schema: [iglu:com.snowplowanalytics.snowplow.storage/snowflake_config/jsonschema/1-0-0][snowflake-schema]

Snowflake configuration is available at dedicated [Snowflake Loader wiki][snowflake-setup].


<a name="elasticsearch" />

### Elasticsearch

Schema: [iglu:com.snowplowanalytics.snowplow.storage/elastic_config/jsonschema/1-0-1][elastic-schema]

1. `name`: a descriptive name for this Snowplow storage target
2. `port`: The port to load. Normally 9200, should be 80 for Amazon Elasticsearch Service.
3. `index`: The Elasticsearch index to load
4. `nodesWanOnly`: if this is set to true, the EMR job will disable node discovery. This option is necessary when using Amazon Elasticsearch Service.
5. `type`: name of type
6. `purpose`: common for all targets. Elasticsearch supports only `FAILED_EVENTS`
7. `id`: optional machine-readable config id

For information on setting up Elasticsearch itself, see [[Setting up Amazon Elasticsearch Service]].

<a name="dynamodb">

### Amazon DynamoDB

Schema: [iglu:com.snowplowanalytics.snowplow.storage/amazon_dynamodb_config/jsonschema/1-0-1][amazon-dynamodb-schema]

1. `name`: a descriptive name for this Snowplow storage target
2. `accessKeyId`: AWS Access Key Id
3. `secretAccessKey`: AWS Secret Access Key
4. `awsRegion`: AWS region
5. `dynamodbTable`: DynamoDB table to store information about processed events
6. `purpose`: common for all targets. Elasticsearch supports only `DUPLICATE_TRACKING`
7. `id`: optional machine-readable config id

[amazon-dynamodb-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow.storage/amazon_dynamodb_config/jsonschema/1-0-1
[elastic-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow.storage/elastic_config/jsonschema/1-0-1
[postgresql-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow.storage/postgresql_config/jsonschema/1-0-1
[redshift-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow.storage/redshift_config/jsonschema/2-1-0
[snowflake-schema]:  https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow.storage/snowflake_config/jsonschema/1-0-0

[redshift-copy]: http://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html

[snowflake-setup]: https://github.com/snowplow-incubator/snowplow-snowflake-loader/wiki/Setup-Guide
[secure-redshift-password]: https://github.com/snowplow/snowplow/wiki/Setting-up-Redshift#secure-password
