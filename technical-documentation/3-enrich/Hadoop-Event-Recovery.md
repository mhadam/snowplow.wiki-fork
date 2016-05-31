[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow technical documentation) > [**Enrichment**](Enrichment) > Hadoop Event Recovery

### Overview

Hadoop Event Recovery allows you to fix Snowplow bad rows and make them ready for reprocessing. It has not yet been released.

You will typically run the Hadoop Event Recovery jar as part of an EMR jobflow with three steps:

1. S3DistCp to move the event files you want to reprocess from S3 to HDFS. This step uses the groupBy option to combine small files into larger blocks, speeding up the job.
2. Hadoop Event Recovery itself. This step uses your custom JavaScript to choose which events to reprocess and how to change them.
3. Another S3DistCp step to move the recovered events from HDFS to S3.

### Writing the JavaScript

You need to write your reprocessing logic in JavaScript. To do this, you need to write a `process` function which takes two arguments: the raw line TSV string and an array of error message strings. It should return null for bad rows which you don't want to reprocess, and a fixed-up raw line TSV string otherwise. You can define other functions besides `process`.

Your JavaScript will be executed using [Rhino](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Rhino).

Changing events using Hadoop Event Recovery requires understanding of the Snowplow raw event format.

We provide several built-in functions which are commonly used by event recovery scripts. The are:

#### tsvToArray

Split the input event TSV into an array

#### arrayToTsv

Turn an array of strings into a TSV

#### parseQuerystring

Convert a querystring into a JavaScript object

#### buildQuerystring

Convert a JavaScript object into a querystring

#### parseJson

Converts a JSON string into a JavaScript object

#### stringifyJson

Converts a JavaScript object into a JSON string. Use this instead of `JSON.stringify`. 

#### decodeBase64

Base64-decodes a string

#### encodeBase64

Base64-encodes a string

### An example

You could use this JavaScript to change the Tracker Version of every bad event to "js-2.7.0":

```javascript
function process(event, errors) {
	var fields = tsvToArray(event);
	var querystringDict = parseQuerystring(fields[11]);
	querystringDict['tv'] = 'js-2.7.0';
	fields[11] = buildQuerystring(querystringDict);
	return arrayToTsv(fields);
}
```

You can find other examples in the [Hadoop Event Recovery test suite](https://github.com/snowplow/snowplow/blob/master/3-enrich/hadoop-event-recovery/src/test/scala/com/snowplowanalytics/hadoop/scalding/SnowplowEventRecoveryJobSpec.scala).

### Encoding the JavaScript

You need to Base64 encode your JavaScript. You can do this using the coreutils base64 program:

```bash
base64 myJsFile
```

Or you can use an online tool like [www.base64encode.org](https://www.base64encode.org/).

### Running the job

You will run Hadoop Event Recovery directly from the command line using the AWS Command Line Interface. An example command:

```bash
aws emr create-cluster --applications Name=Hadoop --ec2-attributes '{
    "InstanceProfile":"EMR_EC2_DefaultRole",
    "AvailabilityZone":"us-east-1d",
    "EmrManagedSlaveSecurityGroup":"sg-2f9aba4b",
    "EmrManagedMasterSecurityGroup":"sg-2e9aba4a"
}' --service-role EMR_DefaultRole --enable-debugging --release-label emr-4.3.0 --log-uri 's3n://{{path to logs}}' --steps '[
{
    "Args":[
        "--src",
        "s3n://{{my-output-bucket/enriched/bad}}/",
        "--dest",
        "hdfs:///local/monthly/",
        "--groupBy",
        ".*201(5-1[12]|6-[0-9][0-9]).*",
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
    "Jar":"s3://snowplow-hosted-assets/3-enrich/hadoop-event-recovery/snowplow-hadoop-event-recovery-0.1.0.jar",
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

Replace the `{{...}}` placeholders above with the appropriate bucket paths.
