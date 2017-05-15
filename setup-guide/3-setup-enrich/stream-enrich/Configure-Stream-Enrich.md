<a name="top" />

[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » [Step 3.2: setting up Stream Enrich](setting-up-stream-enrich) » [[Install Stream Enrich]] » Configure Stream Enrich

This documentation is for version 0.6.0 - 0.9.0 of Stream Enrich. Documentation for other versions is available:

* **[Version 0.1.0][v0.1]**
* **[Version 0.3.0][v0.3]**
* **[Version 0.5.0][v0.5]**

Stream Enrich has a number of configuration options available.

## Basic configuration

### Template

Download a template configuration file from GitHub: [config.hocon.sample][app-conf].

Now open the `config.hocon.sample` file in your editor of choice.

### AWS settings

Values that must be configured are:

+ `enrich.aws.access-key`
+ `enrich.aws.secret-key`

You can insert your actual credentials in these fields. Alternatively, if you set both fields to "env", your credentials will be taken from the environment variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY.

### Source

The `enrich.source` setting determines which of the supported sources to read raw Snowplow events from:

+ `"kinesis`" for reading Thrift-serialized records from a named Amazon Kinesis stream
+ `"stdin`" for reading Base64-encoded Thrift-serialized records from the app's own `stdin` I/O stream

If you select `"kinesis"`, you need to set some other fields in the `enrich.streams.in` section:

* `enrich.streams.in.raw`: the name of your raw Snowplow event stream [configured in your Scala Stream Collector](Configure-the-Scala-Stream-Collector).
* `enrich.streams.in.buffer`
* `enrich.streams.in.buffer`: Stream Enrich maintains a buffer of enriched events and won't send them until certain conditions are met.
 - `buffer.byte-limit`: Whenever the total size of the buffered records exceeds this number, the buffer will be flushed.
 - `buffer.record-limit`: Whenever the total number of buffered records exceeds this number, the buffer will be flushed.
 - `buffer.time-limit`: If this length of time passes without the buffer being flushed, the buffer will be flushed.


If you select `"kinesis"`, you need to set `enrich.streams.in` to the name of your raw Snowplow event stream [configured in your Scala Stream Collector](Configure-the-Scala-Stream-Collector).

### Sinks

The `enrich.sink` setting determines which of the supported sinks to write enriched Snowplow events to:

+ `"kinesis"` for writing enriched Snowplow events to a named Amazon Kinesis stream
+ `"stdouterr"` for writing enriched Snowplow events records to the app's own `stdout` I/O stream

If you select `"kinesis"`, you will also need to update the `enrich.streams.out` section:

```
out: {
  enriched: "SnowplowEnriched"
  bad: "SnowplowBad"
}
```

### Monitoring

You can also now include Snowplow Monitoring in the application.  This is setup through a new section at the bottom of the config.  You will need to ammend:

+ `monitoring.snowplow.collector-uri` insert your snowplow collector URI here.
+ `monitoring.snowplow.app-id` the app-id used in decorating the events sent.

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
