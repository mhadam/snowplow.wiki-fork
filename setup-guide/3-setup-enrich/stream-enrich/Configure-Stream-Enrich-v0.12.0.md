<a name="top" />

[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » [Step 3.2: setting up Stream Enrich](setting-up-stream-enrich) » [[Install Stream Enrich]] » Configure Stream Enrich

This documentation is for version 0.12.0 of Stream Enrich. Documentation for other versions is available:

Stream Enrich has a number of configuration options available.

## Basic configuration

### Template

Download a template configuration file from GitHub: [config.hocon.sample][app-conf].

Now open the `config.hocon.sample` file in your editor of choice.

### AWS settings

Values that must be configured are:

+ `enrich.aws.accessKey`
+ `enrich.aws.secretKey`

You can insert your actual credentials in these fields. Alternatively, if you set both fields to "env", your credentials will be taken from the environment variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY.

## Configuration

The Stream Enrich is configured using a HOCON file. These are the fields:

* `source`: Choose `"kinesis"`, `"nsq"`, `"kafka"` or `"stdin"` as a source stream
* `sink`: Choose between `"kinesis"`, `"nsq"`, `"kafka"` or `"stdin"` as a sink stream for failed events
* `streams.in.raw`: The name of the input stream of the tool which you choose as a source. This should be the stream to which your are writing records with the Scala Stream Collector.
* `streams.out.enriched`: The name of the output stream of the tool which you choose as enriched events' sink. This is stream where the events that were successfully enriched will end up.
* `streams.out.bad`: The name of the output stream of the tool which you choose as bad events' sink. This is stream  where the event that failed enrichment will be stored.
* `streams.out.partitionKey`: Key for determining how the output stream/topic will be partitioned.
* `streams.kinesis.initialPosition`: Where to start reading from the stream the first time the app is run. "TRIM_HORIZON" for as far back as possible, "LATEST" for as recent as possibly, "AT_TIMESTAMP" for after the specified timestamp.
* `streams.kinesis.initialTimestamp`: Timestamp for "AT_TIMESTAMP" initial position
* `streams.kinesis.maxRecords`: Maximum number of records to read per GetRecords call
* `streams.kinesis.region`: The Kinesis region name to use.
* `streams.kinesis.backoffPolicy.minBackoff`: Minimum backoff periods for Kinesis.
* `streams.kinesis.backoffPolicy.maxBackoff`: Maximum backoff periods for Kinesis.
* `streams.appName`: Unique identifier for the app which ensures that if it is stopped and restarted, it will restart at the correct location.
* `streams.kafka.brokers`: Kafka broker address which will be used
* `streams.kafka.retries`: Number of retries to perform before giving up on sending a record
* `streams.nsq.rawChannel`: Channel name for NSQ source
* `streams.nsq.host`: Host name for NSQ tools
* `streams.nsq.port`: HTTP port for nsqd
* `streams.nsq.lookupPort`: HTTP port for nsqlookupd
* `streams.buffer.byteLimit`: Whenever the total size of the buffered records exceeds this number, they will all be sent to Kinesis/Kafka.
* `streams.buffer.recordLimit`: Whenever the total number of buffered records exceeds this number, they will all be sent to Kinesis/Kafka.
* `streams.buffer.timeLimit`: If this length of time passes without the buffer being flushed, the buffer will be flushed.

### Monitoring

You can also now include Snowplow Monitoring in the application.  This is setup through a new section at the bottom of the config.  You will need to ammend:

+ `monitoring.snowplow.collectorUri` insert your snowplow collector URI here.
+ `monitoring.snowplow.collectorPort` insert port number which is Snowplow Collector is running on.
+ `monitoring.snowplow.appId` the app id used in decorating the events sent.
+ `monitoring.snowplow.method` HTTP method for Snowplow Tracking

If you do not wish to include Snowplow Monitoring please remove the entire `monitoring` section from the config.

## Resolver configuration

You will also need a JSON configuration for the Iglu resolver used to look up JSON schemas. A sample configuration is available [here][resolver.json.sample].

### Storage in DynamoDB

Rather than keeping the resolver JSON in a local file, you can store it in a [DynamoDB][ddb] table with hash key "id". If you do this, the JSON must be saved in string form in an item under the key "json".

### Configuring enrichments

You may wish to use Snowplow's configurable enrichments. To do this, create a directory of enrichment JSONs. For each configurable enrichment you wish to use, the enrichments directory should contain a .json file with a configuration JSON for that enrichment. When you come to run Stream Enrich you can then pass in the filepath to this directory using the --enrichments option.

Sensible default configuration enrichments are available on GitHub: [3-enrich/emr-etl-runner/config/enrichments][enrichment-json-examples].

See the documentation on [configuring enrichments][configuring-enrichments] for details on the available enrichments.

### Storage in DynamoDB

Rather than keeping the enrichment configuration JSONs in a local directory, you can store them in DynamoDB in a table with hash key "id". Each JSON should be stored in its own item in the table, under the key "json". The values of the "id" key for these items should have a common prefix so that Stream Enrich can tell which items in the table contain enrichment configuration JSONs.

### GeoLiteCity databases

For the `ip_lookups` enrichment you can manually download the latest version of the [MaxMind GeoLiteCity database][geolite] jarfile directly from our Hosted Assets bucket on Amazon S3 - please see our [[Hosted assets]] page for details.

If you do not have the databases downloaded prior to running but still have the `ip_lookups` enrichment activated then these databases will be downloaded automatically and saved in the working directory. Not that although Spark Enrich can download databases whose URIs begin with "s3://" from S3, Stream Enrich can't.

Next: [[Run Stream Enrich]]

[v0.1]: https://github.com/snowplow/snowplow/wiki/Configure-Scala-Kinesis-Enrich-v0.1
[v0.3]: https://github.com/snowplow/snowplow/wiki/Configure-Scala-Kinesis-Enrich-v0.3
[v0.5]: https://github.com/snowplow/snowplow/wiki/Configure-Scala-Kinesis-Enrich-v0.5
[geolite]: http://dev.maxmind.com/geoip/legacy/geolite/?rld=snowplow
[app-conf]: https://raw.githubusercontent.com/snowplow/snowplow/master/3-enrich/stream-enrich/examples/config.hocon.sample
[enrichment-json-examples]: https://github.com/snowplow/snowplow/tree/master/3-enrich/emr-etl-runner/config/enrichments
[configuring-enrichments]: https://github.com/snowplow/snowplow/wiki/Configurable-enrichments
[resolver.json.sample]: https://raw.githubusercontent.com/snowplow/snowplow/master/3-enrich/config/iglu_resolver.json
[ddb]: http://aws.amazon.com/dynamodb/
