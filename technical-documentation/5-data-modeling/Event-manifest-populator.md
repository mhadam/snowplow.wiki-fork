[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Data modeling**](data-modeling-documentation) > Event Manifest Populator

## Overview

[Event Manifest Populator][event-manifest-populator] is an [Apache Spark][spark] job allowing you to backpopulate a Snowplow event manifest in DynamoDB with the metadata of some or all enriched events from your archive in S3. 

This one-off job solves the "cold start" problem for identifying cross-batch natural deduplicates in Snowplow's [Hadoop Shred step][shredding].
In other words, without running this job you still will be able to deduplicate events across batches, 
but if Hadoop Shred encounter duplicate of event that was shredded *before* you enabled cross-batch deduplication it will land into `shredded/good`.

### Usage

In order to use Event Manifest Populator, you need to have [boto2][boto] and [pyinvoke][pyinvoke] installed:

```
$ pip install boto pyinvoke
```

As next step you need to grab `tasks.py` file with instructions to run job on AWS EMR.
You can do it by downloading it directly from Github:

```
$ wget https://raw.githubusercontent.com/snowplow/snowplow/master/5-data-modeling/event-manifest-populator/tasks.py
```

Now you can run Event Manifest Populator with a single command (inside a directory with `tasks.py`):

```
$ inv run_emr $ENRICHED_ARCHIVE_S3_PATH $STORAGE_CONFIG_PATH $IGLU_RESOLVER_PATH
```

Task has three required arguments: 

1. Path to enriched events archive. It can be found in `aws.s3.buckets.enriched.archive` setting in your [config.yml][config]. It also can be passed as `--enriched-archive` option
2. Local path to [Duplicate storage][dynamodb-config] configuration JSON. It also can be passed as `--storage-config` option
3. Local path to [Iglu resolver][resolver] configuration JSON. It also can be passed as `--resolver` option

Optionally, you can also pass following arguments:

* `--since` to reduce amount of data to be stored in DynamodDB. 
  If this option was passed Manifest Populator will process enriched events only after specified date.
  Input date supports two formats: `YYYY-MM-dd` and `YYYY-MM-dd-HH-mm-ss`.
* `--log-path` to store EMR job logs on S3. Normally, Manifest Populator does not
  produce any logs or output, but if some error occured you'll be able to
  inspect it in EMR logs stored in this path.
* `--profile` to specify AWS profile to create this EMR job.
* `--jar` to specify S3 path to custom JAR


[spark]: http://spark.apache.org/

[boto]: http://boto.cloudhackers.com/en/latest/
[pyinvoke]: http://www.pyinvoke.org/

[event-manifest-populator]: https://github.com/snowplow/snowplow/tree/master/5-data-modeling/event-manifest-populator/
[tasks-py]: https://raw.githubusercontent.com/snowplow/snowplow/master/5-data-modeling/event-manifest-populator/tasks.py
[config]: https://github.com/snowplow/snowplow/blob/master/3-enrich/emr-etl-runner/config/config.yml.sample
[resolver]: https://github.com/snowplow/iglu/wiki/Iglu-client-configuration
[shredding]: https://github.com/snowplow/snowplow/wiki/Shredding

[dynamodb-config]: https://github.com/snowplow/snowplow/wiki/Configuring-storage-targets#dynamodb

