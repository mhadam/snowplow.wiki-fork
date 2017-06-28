[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Storage**](storage-documentation) > Kinesis LZO S3 Sink

## Overview

The Kinesis LZO S3 Sink reads records from an [Amazon Kinesis][kinesis] stream. The records are
serialized in [Thrift][thrift] using [Elephant Bird's][elephant-bird] `RawBlockWriter` class, then
compressed using [LZO][hadoop-lzo]. This generates .lzo files (the compressed data) and small
.lzo.index files (containing byte offsets for the LZO blocks in the corresponding .lzo files,
allowing the blocks to be processed in parallel).

See also the [setup guide][setup-guide].

[kinesis]: http://aws.amazon.com/kinesis/
[snowplow]: http://snowplowanalytics.com
[hadoop-lzo]: https://github.com/twitter/hadoop-lzo
[thrift]: https://github.com/apache/thrift
[s3]: http://aws.amazon.com/s3/
[elephant-bird]: https://github.com/twitter/elephant-bird/
[setup-guide]: https://github.com/snowplow/snowplow/wiki/kinesis-lzo-s3-sink-setup
