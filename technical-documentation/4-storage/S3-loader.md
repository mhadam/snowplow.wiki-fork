[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Storage**](storage-documentation) > Snowplow S3 Loader

## Overview

The Snowplow S3 Loader consumes records from an [Amazon Kinesis][kinesis] stream or [NSQ][nsq] topic, and writes them to [S3][s3].

There are 2 file formats supported:
 * LZO
 * GZip

### LZO

The records are treated as raw byte arrays. [Elephant Bird's][elephant-bird] `BinaryBlockWriter` class is used to serialize them as a [Protocol Buffers][protobufs] array (so it is clear where one record ends and the next begins) before compressing them.

The compression process generates both compressed .lzo files and small .lzo.index files ([splittable LZO][hadoop-lzo]). Each index file contain the byte offsets of the LZO blocks in the corresponding compressed file, meaning that the blocks can be processed in parallel.

### GZip

The records are treated as byte arrays containing UTF-8 encoded strings (whether CSV, JSON or TSV). New lines are used to separate records written to a file. This format can be used with the Snowplow Kinesis Enriched stream, among other streams.

See also the [setup guide](snowplow-s3-loader-setup).

[kinesis]: http://aws.amazon.com/kinesis/
[nsq]: http://nsq.io/
[hadoop-lzo]: https://github.com/twitter/hadoop-lzo
[s3]: http://aws.amazon.com/s3/
[elephant-bird]: https://github.com/twitter/elephant-bird/
[protobufs]: https://github.com/google/protobuf/
