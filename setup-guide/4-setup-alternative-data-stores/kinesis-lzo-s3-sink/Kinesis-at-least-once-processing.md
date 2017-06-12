[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 4: setting up alternative data stores**](Setting-up-alternative-data-stores) » [Kinesis-LZO-S3-Sink-Setup](Kinesis-LZO-S3-Sink-Setup) » Kinesis at-least-once processing

## Overview

Kinesis LZO S3 Sink is using the [Amazon Kinesis Client Library](http://docs.aws.amazon.com/kinesis/latest/dev/developing-consumers-with-kcl.html#kinesis-record-processor-overview-kcl) (KCL). The KCL helps to consume and process data from Amazon Kinesis streams. It takes care of many of the complex tasks associated with distributed computing, such as load-balancing across multiple instances, responding to instance failures, checkpointing processed records, and reacting to resharding.

On the first run of the application, the Kinesis Connectors Library creates a DynamoDB table to keep track of what it has consumed from the stream so far. Either on first run it starts consuming from `LATEST` being the most recent record in the stream or `TRIM_HORIZON` being the oldest record available in the stream.

At runtime:

- Kinesis data processing is ordered per partition and occurs at-least-once per message.
- Multiple applications can read from the same Kinesis stream. Kinesis will maintain the application-specific shard and checkpoint info in DynamoDB.
- A single Kinesis stream shard is processed by one Kinesis consumer at a time.
- A single Kinesis consumer can read from multiple shards of a Kinesis stream.
- Each Kinesis consumer maintains its own checkpoint info.

## Kinesis Checkpointing

Kinesis checkpointing works in the following way:

- Each Kinesis consumer periodically stores the current position of the stream in the backing DynamoDB table. This allows the system to recover from failures and continue processing where the application left off.
- If no Kinesis checkpoint info exists when the consumer starts, it will start either from the oldest record available (`TRIM_HORIZON`) or from the latest tip (`LATEST`). This is configurable.
- `LATEST` could lead to missed records if data is added to the stream while no Kinesis consumer is running (and no checkpoint info is being stored).
- `TRIM_HORIZON` may lead to duplicate processing of records where the impact is dependent on checkpoint frequency and processing idempotency.

The DynamoDB table is updated every time that events are successfully sent to S3. Thus, if the Kinesis sink, say, goes down for whatever reason just after processing data and just before it was able to update DynamoDB (a very small amount of time) you might end up with a small amount of duplication when it is restarted.

In other words, streams have "**at least once**" semantics, meaning that every data record from a shard is processed at least one time by a worker in Kinesis consumer. This ensures that no data is lost.

We are steadily introducing de-duplication into our Redshift loading process, with the goal that our data in Redshift is dupe-free. We can explore these techniques for other data stores too.

### Further reading on deduplication

- [Dealing with duplicate event IDs](http://snowplowanalytics.com/blog/2015/08/19/dealing-with-duplicate-event-ids/)
- [Event fingerprint enrichment](Event-fingerprint-enrichment)
- [Event de-duplication in Relational Database Shredder](http://snowplowanalytics.com/blog/2016/01/26/snowplow-r76-changeable-hawk-eagle-released/#deduplication)
