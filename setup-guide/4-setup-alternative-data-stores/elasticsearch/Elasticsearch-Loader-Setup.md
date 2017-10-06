<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 4: setting up alternative data stores**](Setting-up-alternative-data-stores) > [Elasticsearch-Loader-Setup](Elasticsearch-Loader-Setup)

This documentation is for version 0.10.0 and 0.10.1 of the Elasticsearch Loader.  For previous
versions, refer to:

* **[Version 0.1.0 - 0.3.0][0.1]**
* **[Version 0.4.0][0.4]**
* **[Version 0.5.0 - 0.6.0][0.5]**
* **[Version 0.7.0][0.7]**
* **[Version 0.8.0][0.8]**
* **[Version 0.9.0][0.9]**

## Overview

If you are using [Stream Enrich][stream-enrich] to write enriched Snowplow events to one stream and bad events to another, you can use the Elasticsearch Loader to read events from either of those streams and write them to [Elasticsearch][elasticsearch].

## Configuring Elasticsearch

### Getting started

First off, install and set up Elasticsearch version 5.x or 2.x.x. For more information check out the [installation guide][installation-guide].

### Raising the file limit

Elasticsearch keeps a lot of files open simultaneously so you will need to increase the maximum number of files a user can have open. To do this:

```bash
sudo vim /etc/security/limits.conf
```

Append the following lines to the file:

```bash
{{USERNAME}} soft nofile 32000
{{USERNAME}} hard nofile 32000
```

Where {{USERNAME}} is the name of the user running Elasticsearch. You will need to logout and restart Elasticsearch before the new file limit takes effect.

To check that this new limit has taken effect you can run the following command from the terminal:

```bash
curl localhost:9200/_nodes/process?pretty
```

If the `max_file_descriptors` equals 32000 it is running with the new limit.

### Defining the mapping

Use the following request to create the mapping for the enriched event type:

```bash
curl -XPUT 'http://localhost:9200/snowplow' -d '{
    "settings": {
        "analysis": {
            "analyzer": {
                "default": {
                    "type": "keyword"
                }
            }
        }
    },
    "mappings": {
        "enriched": {
            "_ttl": {
              "enabled":true,
              "default": "604800000"
            },
            "properties": {
                "geo_location": {
                    "type": "geo_point"
                }
            }
        }
    }
}'
```

Elasticsearch will then treat the collector_tstamp field as the timestamp and the geo_location field
as a "geo_point". Documents will be automatically deleted one week (604800000 milliseconds) after
their collector_tstamp.

This initialization sets the default analyzer to "keyword". This means that string fields will not
be split into separate tokens for the purposes of searching. This saves space and ensures that URL
fields are handled correctly.

If you want to tokenize specific string fields, you can change the "properties" field in the mapping
like this:

```bash
curl -XPUT 'http://localhost:9200/snowplow' -d '{
    "settings": {
        "analysis": {
            "analyzer": {
                "default": {
                    "type": "keyword"
                }
            }
        }
    },
    "mappings": {
        "enriched": {
            "_timestamp" : {
                "enabled" : "yes",
                "path" : "collector_tstamp"
            },
            "_ttl": {
              "enabled":true,
              "default": "604800000"
            },
            "properties": {
                "geo_location": {
                    "type": "geo_point"
                },
                "field_to_tokenize": {
                    "type": "string",
                    "analyzer": "english"
                }
            }
        }
    }
}'
```

## Installing the Elasticsearch Loader

You can choose to either:

1. Download the Elasticsearch Loader jarfile, _or:_
2. Compile it from source

### Downloading the jarfile

To get a local copy, you can download the executable jarfile directly from our Hosted Assets bucket
on Amazon S3 - please see our [[Hosted assets]] page for details.

### Compiling from source

Alternatively, you can build it from the source files. To do so, you will need [scala][scala] and [sbt][sbt] installed.

To do so, clone the Elasticsearch loader repo:

```bash
git clone https://github.com/snowplow/snowplow-elasticsearch-loader.git
```

Use `sbt` to resolve dependencies, compile the source, and build a fat JAR file with all
dependencies.

```bash
sbt "project http" assembly # if you want to use the HTTP API compatible with every ES versions.
sbt "project tcp" assembly # if you want to use the transport API with a 5.x cluster
sbt "project tcp2x" assembly # if you want to use the transport API with a 2.x cluster
```

You will then find the fat jar in the corresponding directory:
`{http,tcp,tcp2x}/target/scala-2.11/snowplow-elasticsearch-loader-{http,tcp,tcp-2x}-0.10.0.jar`.
It is now ready to be deployed.

## Using the Elasticsearch Loader

### Configuration

The sink is configured using a HOCON file, for which you can find an example [here][config-file].
These are the fields:

* `source`: Change this to "kinesis", "stdin" or "nsq".
* `sink.good`: Where to write good events. "elasticsearch" or "stdout".
* `sink.bad`: Where to write error JSONs for bad events. "kinesis", "stderr" or "nsq" (or "none" to ignore bad events).
* `enabled`: "good" if the input stream contains successfully enriched events; "bad" if it contains bad rows; "plain-json" if it contains plain JSON.
* `aws.accessKey` and `aws.secretKey`: Change these to your AWS credentials. You can alternatively leave them as "default", in which case the [DefaultAWSCredentialsProviderChain][DefaultAWSCredentialsProviderChain] will be used.
* `kinesis.initialPosition`: Where to start reading from the stream the first time the app is run. "TRIM_HORIZON" for as far back as possible, "LATEST" for as recent as possibly, "AT_TIMESTAMP" for after specified timestamp.
* `kinesis.initialTimestamp`: Need to be specified when initial position is "AT_TIMESTAMP"
* `kinesis.maxRecords`: Maximum number of records fetched in a single request.
* `kinesis.region`: The AWS region where the streams are located.
* `kinesis.appName`: Unique identifier for the app which ensures that if it is stopped and restarted, it will restart at the correct location.
* `nsq.channelName`: Channel name for NSQ source stream. If more than one application reading from the same NSQ topic at the same time, all of them must have unique channel name to be able to get all the data from the same topic
* `nsq.host`: Hostname for NSQ tools
* `nsq.port`: HTTP port number for nsqd
* `nsq.lookupPort`: HTTP port number for nsqlookupd
* `stream.inStreamName`: The name of the input stream of the tool which you choose as a source.
* `stream.outStreamName`: The name of the output stream of the tool which you choose as sink. Records which cannot be converted to JSON or can be converted but are rejected by Elasticsearch get sent here.
* `streams.buffer.byteLimit`: Whenever the total size of the buffered records exceeds this number, they will all be sent to elasticsearch. 
* `streams.buffer.recordLimit`: Whenever the total number of buffered records exceeds this number, they will all be sent to elasticsearch.
* `streams.buffer.timeLimit`: If this length of time passes without the buffer being flushed, the buffer will be flushed.
* `elasticsearch.client.endpoint`: The Elasticesarch cluster endpoint.
* `elasticsearch.client.port`: The Elasticesarch cluster port.
* `elasticsearch.client.maxTimeout`: The Elasticesarch maximum timeout in milliseconds.
* `elasticsearch.client.ssl`: If using the HTTP API whether to use SSL or not.
* `elasticsearch.aws.signing`: If using the Amazon Elasticsearch service and the HTTP API, this lets
you sign your requests.
* `elasticsearch.aws.region`: If signing API requests, region where the Elasticsearch cluster is
located.
* `elasticsearch.cluster.name`: The Elasticesarch cluster name.
* `elasticsearch.cluster.index`: The Elasticsearch index name.
* `elasticsearch.cluster.clusterType`: The Elasticesarch type name.

### Monitoring

You can also include Snowplow Monitoring in the application.  This is set up through a new section at the bottom of the config.  You will need to ammend:

+ `monitoring.snowplow.collectorUri` insert your snowplow collector URI here.
+ `monitoring.snowplow.appId` the app-id used in decorating the events sent.

If you do not wish to include Snowplow Monitoring, remove the entire `monitoring` section from the config.

### Execution

The Elasticsearch Loader is a jarfile. Simply provide the configuration file as a parameter:

```bash
java -jar snowplow-elasticsearch-loader-http-0.10.0.jar --config my.conf # if using the HTTP API
java -jar snowplow-elasticsearch-loader-tcp-0.10.0.jar --config my.conf # if using the transport API with a 5.x cluster
java -jar snowplow-elasticsearch-loader-tcp-2x-0.10.0.jar --config my.conf # if using the transport API with a 2.x cluster
```

This will start the process of reading events from Kinesis and writing them to an Elasticsearch cluster.

[elasticsearch]: http://www.elasticsearch.org/overview/
[installation-guide]: https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html
[DefaultAWSCredentialsProviderChain]: http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html
[stream-enrich]: https://github.com/snowplow/snowplow/wiki/Stream-Enrich
[config-file]: https://raw.githubusercontent.com/snowplow/snowplow-elasticsearch-loader/master/examples/config.hocon.sample
[0.1]: https://github.com/snowplow/snowplow/wiki/kinesis-elasticsearch-sink-setup-0.1.0
[0.4]: https://github.com/snowplow/snowplow/wiki/kinesis-elasticsearch-sink-setup-0.4.0
[0.5]: https://github.com/snowplow/snowplow/wiki/kinesis-elasticsearch-sink-setup-0.5.0
[0.7]: https://github.com/snowplow/snowplow/wiki/kinesis-elasticsearch-sink-setup-0.7.0
[0.8]: https://github.com/snowplow/snowplow/wiki/kinesis-elasticsearch-sink-setup-0.8.0
[0.9]: https://github.com/snowplow/snowplow/wiki/kinesis-elasticsearch-sink-setup-0.9.0

[scala]: https://www.scala-lang.org
[sbt]: http://www.scala-sbt.org
