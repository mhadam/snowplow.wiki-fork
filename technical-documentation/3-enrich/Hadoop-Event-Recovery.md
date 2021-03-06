[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Enrichment**](Enrichment) > Hadoop Event Recovery

### Overview

Hadoop Event Recovery allows you to fix Snowplow bad rows and make them ready for reprocessing.

You will typically run the Hadoop Event Recovery jar as part of an EMR jobflow with three steps:

1. S3DistCp to move the event files you want to reprocess from S3 to HDFS. This step uses the groupBy option to combine small files into larger blocks, speeding up the job.
2. Hadoop Event Recovery itself. This step uses your custom JavaScript to choose which events to reprocess and how to change them.
3. Another S3DistCp step to move the recovered events from HDFS to S3.

### The Snowplow bad row format

When a raw event fails Spark Enrich validation, it is written out to a bad rows bucket in a format which looks something like this:

```
{
  "line": "2015-06-09\t09:56:09\tNRT12\t831\t211.14.8.250\tGET...",
  "errors": [{
    "level": "error",
    "message": "Could not find schema with key iglu:com.acme/myevent/jsonschema/1-0-0 in any repository"
  }]
}
```

The `line` field is the original TSV input to the enrichment process; the `errors` field is an array of errors describing why the event failed validation.

For events sent by GET request, the format of the TSV is based on Cloudfront's [Web Distribution Log File Format](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/AccessLogs.html). For events sent by POST request, the format is the same, but with an additional field at the end containing the body of the POST request.

The Scala Hadoop Bad Rows jar lets you extract the `line` field and mutate it using custom JavaScript to fix whatever was wrong with it so you can reprocess it by running Spark Enrich again.

### Writing the JavaScript

You need to write your reprocessing logic in JavaScript. To do this, you need to write a `process` function which takes two arguments: the raw line TSV string and an array of error message strings. It should return null for bad rows which you don't want to reprocess, and a fixed-up raw line TSV string otherwise. You can define other functions besides `process`.

Your JavaScript will be executed using [Rhino](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Rhino).

Changing events using Hadoop Event Recovery requires understanding of the Snowplow raw event format.

We provide several built-in functions which are commonly used by event recovery scripts. The are:

#### tsvToArray

Input: String
Output: Array of strings

Split the input event TSV into an array

#### arrayToTsv

Input: Array of strings
Output: String

Turn an array of strings into a TSV

#### parseQuerystring

Input: String
Output: Dictionary wih string keys and string values

Convert a querystring into a JavaScript object

#### buildQuerystring

Input: Dictionary wih string keys and string values
Output: String

Convert a JavaScript object into a querystring

#### parseJson

Input: String
Output: Object

Converts a JSON string into a JavaScript object

#### stringifyJson

Input: Object
Output: String

Converts a JavaScript object into a JSON string. Use this instead of `JSON.stringify`.

#### decodeBase64

Input: String
Output: String

Base64-decodes a string

#### encodeBase64

Input: String
Output: String

Base64-encodes a string

### Examples

You could use this JavaScript to change the Tracker Version of every bad event to "js-2.7.0" for GET requests, and leave POST requests unchanged:

```javascript
function process(event, errors) {
    var fields = tsvToArray(event);
    if (fields[5] == 'GET') {
	    var querystringDict = parseQuerystring(fields[11]);
        querystringDict['tv'] = 'js-2.7.0';
        fields[11] = buildQuerystring(querystringDict);
    }
    return arrayToTsv(fields);
}
```

This script will select bad rows which only failed due to a missing schema, and return the original raw event unchanged:

```javascript
function process(event, errors) {
    for (var i = 0; i < errors.length; i++) {
        if (! /Could not find schema with key/.test(errors[i])) {
            return null;
        }
    }
    return event;
}
```

Now for a more complicated example. Suppose that you again want to reprocess bad events which failed due to a missing schema. But you also have some events which fail validation due to having noncompliant URLs. You could use this JavaScript to replace the bad URLs so those events will be valid again:

```javascript
function process(event, errors) {

    var failedUrl = false;

    for (var i = 0; i < errors.length; i++) {
            var err = errors[i];
            if (isBadUrlError(err)) {
                    failedUrl = true;
            } else if (!isMissingSchemaError(err)) {
                    return null;
            }
    }

    if (failedUrl) {
        var fields = tsvToArray(event);
        fields[9] = 'http://www.placeholder.com';

        if (fields[5] == 'GET') {

                var querystring = parseQuerystring(fields[11]);
                querystring['url'] = 'http://www.placeholder.com';
                querystring['refr'] = 'http://www.placeholder.com';
                fields[11] = buildQuerystring(querystring);
                return arrayToTsv(fields);
        } else if (fields[5] == 'POST') {

                var postPosition = fields.length - 1;
                var decodedPost = decodeBase64(fields[postPosition]);
                var postJson = parseJson(decodedPost);
                for (var i = 0; i < postJson.data.length; i++) {
                        postJson.data[i].url = 'http://www.placeholder.com';
                        postJson.data[i].refr = 'http://www.placeholder.com';
                }
                fields[postPosition] = encodeBase64(stringifyJson(postJson));
                return arrayToTsv(fields);
        } else {
                return null;
        }
    } else {
        return event;
    }
}

function isBadUrlError(err) {
    return /RFC 2396|could not be parsed by Netaporter|Unexpected error creating URI from string/.test(err);
}

function isMissingSchemaError(err) {
    return /Could not find schema with key/.test(err);
}
```

You can find other examples in the [Hadoop Event Recovery test suite](https://github.com/snowplow/snowplow/blob/master/3-enrich/hadoop-event-recovery/src/test/scala/com/snowplowanalytics/hadoop/scalding/SnowplowEventRecoveryJobSpec.scala).

### Encoding the JavaScript

You need to Base64 encode your JavaScript. You can do this using the coreutils base64 program:

```bash
base64 myJsFile
```

Or you can use an online tool like [www.base64encode.org](https://www.base64encode.org/).

### Restricting the input

The bad rows bucket is organized like this:

```
s3://path/to/bad/
    run=2015-10-07-15-25-53/
        part-00000
        part-00001
        part-00002
        part-00003
    run=2015-10-08-15-25-53/
        part-00000
        part-00001
        part-00002
        part-00003
    run=2015-10-09-15-25-53/
        part-00000
        part-00001
        part-00002
        part-00003
```

All the bad rows created by a given enrichment job end up in the same bucket, and that bucket is timestamped based on when the job was begun.

You will use [S3DistCp](http://docs.aws.amazon.com/ElasticMapReduce/latest/DeveloperGuide/UsingEMR_s3distcp.html) to select the bad rows you wish to process and copy them onto the Hadoop cluster. S3DistCp takes an `--src` option defining the location of the data to copy, but you will just pass "s3://path/to/enriched/bad/*" to this option and use the `--groupBy` option to restrict the job's input to a subset of the bad rows bucket.

`--groupBy` allows you to write a regular expression with a single parenthesized capturing group. Any input paths which don't match the regular expression will be ignored; moreover, input files whose paths have the same value for the parenthesized capturing group will be concatenated. This reduces the total number of files and speeds up the job. For example, you could use the regular expression

```
.*run=201(5-1[12]|6-[0-9][0-9]).*
```

to only process inputs dating from between November 2015 and December 2016. This regex would also cause input files from the same month to be concatenated (since the brackets go around only the part of the regex corresponding to the year and month).

We also use the `--targetSize` argument to control the maximum size up to which files are concatenated. Because `--targetSize` limits the total size of the concatenated files, there is no reason not to write the `--groupBy` regex in such a way that all files end up in a single group, as we do in the following examples:

To process all files from 2014, concatenating all files:

```
.*(run)=2014.*
```

To process only files from the 24th of February, 2016:

```
.*(run)=2016-02-24.*
```

To process all files from between 3rd January 2016 and 10th August 2016 inclusive:

```
.*(run)=2016-(?:01-(?:0[3-9]|[1-3][0-9])|0[2-7]-[0-9][0-9]|08-0[0-9]|08-10).*
```

[Debuggex](https://www.debuggex.com/) makes building these regular expressions more intuitive.

### Running the job

You will run Hadoop Event Recovery directly from the command line using the AWS Command Line Interface. An example command:

```bash
aws emr create-cluster --applications Name=Hadoop --ec2-attributes '{
    "InstanceProfile":"EMR_EC2_DefaultRole",
    "AvailabilityZone":"{{...}}",
    "EmrManagedSlaveSecurityGroup":"{{...}}",
    "EmrManagedMasterSecurityGroup":"{{...}}"
}' --service-role EMR_DefaultRole --enable-debugging --release-label emr-4.3.0 --log-uri 's3n://{{path to logs}}' --steps '[
{
    "Args":[
        "--src",
        "s3n://{{my-output-bucket/enriched/bad}}/",
        "--dest",
        "hdfs:///local/monthly/",
        "--groupBy",
        ".*(run)=2014.*",
        "--targetSize",
        "128",
        "--outputCodec",
        "lzo"
    ],
    "Type":"CUSTOM_JAR",
    "ActionOnFailure":"TERMINATE_CLUSTER",
    "Jar":"/usr/share/aws/emr/s3-dist-cp/lib/s3-dist-cp.jar",
    "Name":"Combine Months"
},
{
    "Args":[
        "com.snowplowanalytics.hadoop.scalding.SnowplowEventRecoveryJob",
        "--hdfs",
        "--input",
        "hdfs:///local/monthly/*",
        "--output",
        "hdfs:///local/recovery/",
        "--inputFormat",
        "bad",
        "--script",
        "ZnVuY3Rpb24gcHJvY2VzcyhldmVudCwgZXJyb3JzKSB7CiAgICAvLyBPbmx5IHJlcHJvY2VzcyBpZjoKICAgIC8vIDEuIHRoZXJlIGlzIG9ubHkgb25lIHZhbGlkYXRpb24gZXJyb3IgYW5kCiAgICAvLyAyLiB0aGUgZXJyb3IgcmVmZXJlbmNlcyBSRkMgMjM5Niwgd2hpY2ggc3BlY2lmaWVzIHdoYXQgbWFrZXMgYSBVUkwgdmFsaWQuCiAgICBpZiAoZXJyb3JzLmxlbmd0aCA8IDIgJiYgL1JGQyAyMzk2Ly50ZXN0KGVycm9yc1swXSkpIHsKICAgICAgICB2YXIgZmllbGRzID0gdHN2VG9BcnJheShldmVudCk7CiAgICAgICAgZmllbGRzWzldID0gJ2h0dHA6Ly93d3cucGxhY2Vob2xkZXIuY29tJ1w7CiAgICAgICAgcmV0dXJuIGFycmF5VG9Uc3YoZmllbGRzKTsKICAgIH0gZWxzZSB7CiAgICAgICAgcmV0dXJuIG51bGw7CiAgICB9Cn0K"
    ],
    "Type":"CUSTOM_JAR",
    "ActionOnFailure":"CONTINUE",
    "Jar":"s3://snowplow-hosted-assets/3-enrich/hadoop-event-recovery/snowplow-hadoop-event-recovery-0.2.0.jar",
    "Name":"Fix up bad rows"
},
{
    "Args":[
        "--src",
        "hdfs:///local/recovery/",
        "--dest",
        "s3n://{{my-recovery-bucket/recovered}}"
    ],
    "Type":"CUSTOM_JAR",
    "ActionOnFailure":"TERMINATE_CLUSTER",
    "Jar":"/usr/share/aws/emr/s3-dist-cp/lib/s3-dist-cp.jar",
    "Name":"Back to S3"
}
]' --name 'MyCluster' --instance-groups '[
    {
        "InstanceCount":1,
        "InstanceGroupType":"MASTER",
        "InstanceType":"m1.medium",
        "Name":"MASTER"
    },
    {
        "InstanceCount":2,
        "InstanceGroupType":"CORE",
        "InstanceType":"m1.medium",
        "Name":"CORE"
    }
]'
```

Replace the `{{...}}` placeholders above with the appropriate values, and alter the `--groupBy` regular expression to process the time range of files which you are interested in.

### Checking the result

If the job runs without errors, the fixed-up raw events will be available in `s3://my-recovery-bucket/recovered` for reprocessing.

### Next steps

Assuming your JavaScript was correct, the data in the reprocessing bucket is now in the Snowplow raw event format.

Before proceeding, check that you have your regular job is not running (or about to start). Disable any relevant crons.

Create a copy of your regular `config.yml` file, calling it `config-recovery.yml` or similar. You need to update the `aws.s3.buckets.raw section` of your EmrEtlRunner configuration file so it looks something like this:

```yaml
raw:
  in:
    - s3://does-not-exist # The "in" section will be ignored
  processing: s3://{{my-recovery-bucket/recovered}}
```

We are going to run EmrEtlRunner specifying the reprocessing bucket as the processing bucket, not the in bucket, and we will run EmrEtlRunner with `--skip staging`. This is important - we **must** skip the staging step because that has some specific logic around filenames generated by the Clojure Collector which isn't valid in this case.

Now run EmrEtlRunner with the `--skip staging` option (see [[Setting up EmrEtlRunner]] for more information).

This pipeline run should complete fine - take a note of the run= ID of the recovery run (as seen in the archive folders) so that you can distinguish this run from regular pipeline runs in the future.

And that's it - we have successfully recovered some Snowplow events which were previously "stuck" in our bad rows bucket.

### Caveats

Reprocessing bad rows in this way can cause duplicate events in a couple of ways.

First, an enrichment job can fail partway through having written out some bad rows. When the job is run again, those bad rows will be duplicated, and if you run Scala Hadoop Bad Rows on the output of both jobs, you will end up with duplicate raw events.

Second, a bad row may contain a POST request with multiple events in it, of which not all are necessarily bad. If you run Hadoop Event Recovery on such a POST request and write out all the events it contains, you will end up with duplicate events.
