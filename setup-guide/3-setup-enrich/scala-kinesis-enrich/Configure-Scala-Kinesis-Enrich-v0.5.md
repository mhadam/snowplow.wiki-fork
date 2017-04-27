<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 3: Setting up Enrich**](Setting-up-enrich) > [**Step 3.2: setting up Scala Kinesis Enrich**](Setting-up-Scala-Kinesis-Enrich) > [2: Configuring](Configuring-Scala-Kinesis-Enrich)

This documentation is for version 0.5.0 of Scala Kinesis Enrich. Documentation for other versions is available:

**[Version 0.1.0][v0.1]**
**[Version 0.3.0][v0.3]**

Scala Kinesis Enrich has a number of configuration options available.

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

If you select `"kinesis"`, you need to set `enrich.streams.in` to the name of your raw Snowplow event stream [configured in your Scala Stream Collector](Configure-the-Scala-Stream-Collector).

### Sinks

The `enrich.sink` setting determines which of the supported sinks to write enriched Snowplow events to:

+ `"kinesis"` for writing enriched Snowplow events to a named Amazon Kinesis stream
+ `"stdouterr"` for writing enriched Snowplow events records to the app's own `stdout` I/O stream

If you select `"kinesis"`, you will also need to update the `enrich.streams.out` section:

```
out: {
  enriched: "SnowplowEnriched"
  enriched_shards: 1 # Number of shards to use if created.
  bad: "SnowplowBad" # Not used until #463
  bad_shards: 1 # Number of shards to use if created.
}
```

## Resolver configuration

You will also need a JSON configuration for the Iglu resolver used to look up JSON schemas. A sample configuration is available [here][resolver.json.sample].

### Storage in DynamoDB

Rather than keeping the resolver JSON in a local file, you can store it in a [DynamoDB][ddb] table with hash key "id". If you do this, the JSON must be saved in string form in an item under the key "json".

 ## Configuring enrichments

You may wish to use Snowplow's configurable enrichments. To do this, create a directory of enrichment JSONs. For each configurable enrichment you wish to use, the enrichments directory should contain a .json file with a configuration JSON for that enrichment. When you come to run Scala Kinesis Enrich you can then pass in the filepath to this directory using the --enrichments option.

Sensible default configuration enrichments are available on GitHub: [3-enrich/emr-etl-runner/config/enrichments][enrichment-json-examples].

See the documentation on [configuring enrichments][Configurable-enrichments] for details on the available enrichments.

### Storage in DynamoDB

Rather than keeping the enrichment configuration JSONs in a local directory, you can store them in DynamoDB in a table with hash key "id". Each JSON should be stored in its own item in the table, under the key "json". The values of the "id" key for these items should have a common prefix so that Scala Kinesis Enrich can tell which items in the table contain enrichment configuration JSONs.

### GeoLiteCity databases

For the `ip_lookups` enrichment you can manually download the latest version of the [MaxMind GeoLiteCity database][geolite] jarfile directly from our Hosted Assets bucket on Amazon S3 - please see our [[Hosted assets]] page for details.

If you do not have the databases downloaded prior to running but still have the `ip_lookups` enrichment activated then these databases will be downloaded automatically and saved in the working directory. Not that although Scala Hadoop Enrich can download databases whose URIs begin with "s3://" from S3, Scala Kinesis Enrich can't.

Next: [[Run Scala Kinesis Enrich]]

[v0.1]: https://github.com/snowplow/snowplow/wiki/Configure-Scala-Kinesis-Enrich-v0.1
[v0.3]: https://github.com/snowplow/snowplow/wiki/Configure-Scala-Kinesis-Enrich-v0.3
[geolite]: http://dev.maxmind.com/geoip/legacy/geolite/?rld=snowplow
[app-conf]: https://github.com/snowplow/snowplow/blob/r65-scarlet-rosefinch/3-enrich/scala-kinesis-enrich/src/main/resources/config.hocon.sample
[enrichment-json-examples]: https://github.com/snowplow/snowplow/tree/master/3-enrich/emr-etl-runner/config/enrichments
[configuring-enrichments]: https://github.com/snowplow/snowplow/wiki/5-Configuring-enrichments#template
[resolver.json.sample]: https://github.com/snowplow/snowplow/blob/master/3-enrich/scala-kinesis-enrich/src/main/resources/resolver.json.sample
[ddb]: http://aws.amazon.com/dynamodb/
