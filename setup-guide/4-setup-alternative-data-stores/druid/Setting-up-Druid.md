<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 4: setting up alternative data stores**](Setting-up-alternative-data-stores) > Setup Druid

We support loading Snowplow events into [Druid][druid] as part of the Snowplow Kinesis pipeline.

We have two setup guides for working with Druid:

1. [**Single-instance setup guide**](Setting-up-Druid-single-instance) for getting a local version of Druid working with Snowplow. Useful for development and testing
2. [**AWS production setup guide**](Setting-up-Druid-for-production) for getting a horizontally-scaleable, distributed installation of Druid working with a production Snowplow Kinesis pipeline in AWS

Both guides assume that you are using **Druid 0.8.3 Stable** - this is the version that we have tested these instructions with.

TODO: move the below into the instance spec description

<a name="prereq-java7" />

### 1.4 Java 7 of higher

Each of your cluster machines must be running Java 7 or higher:

```
$ java -version
java version "1.7.0_67"
Java(TM) SE Runtime Environment (build 1.7.0_67-b01)
Java HotSpot(TM) 64-Bit Server VM (build 24.65-b04, mixed mode)
```

[emr-etl-runner]: 1-Installing-EmrEtlRunner
[storage-loader]: 1-Installing-the-StorageLoader
[sql-workbench-tutorial]: http://docs.aws.amazon.com/redshift/latest/gsg/getting-started.html
[redshift-table-def]: https://github.com/snowplow/snowplow/blob/master/4-storage/redshift-storage/sql/atomic-def.sql

[zookeeper]: https://en.wikipedia.org/wiki/Apache_ZooKeeper
[zookeeper-oreilly]: http://shop.oreilly.com/product/0636920028901.do

