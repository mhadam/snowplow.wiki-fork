[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 2c: Setup a Data Store Source**](Setting-up-a-Data-Store-Source) » **AWS Lambda source setup**

## Overview

This tool builds into a single zip file which can be uploaded as a new [AWS Lambda](http://docs.aws.amazon.com/lambda/latest/dg/welcome.html) function. The description you give your Lambda will be used as your collector endpoint (an URL). This means the Lambda you create will need the ability to perform `lambda:GetFunctionConfiguration` on itself, amongst the other normal AWS lambda permissions.

## Deployment guide

This deployment assumes you have the [AWS-CLI](http://docs.aws.amazon.com/cli/latest/userguide/installing.html), and Python ~2.7 installed.

```{bash}
sudo pip install pyyaml
wget https://bintray.com/artifact/download/snowplow/snowplow-generic/snowplow_aws_lambda_source_0.1.0_bundle.zip
unzip snowplow_aws_lambda_source_0.1.0_bundle.zip -d snowplow_aws_lambda_source_0.1.0_bundle
cd snowplow_aws_lambda_source_0.1.0_bundle
```

At this point, fire up your favourite editor and edit the file `config.yaml` (see below).

### Editing `config.yaml`

This configures your deployment. The configuration file is written in [YAML](http://www.yaml.org/spec/1.2/spec.html) and looks as below:

```{yaml}
snowplow:
    collector: http://a-collector.com
    app_id: your_app_id
s3:
    buckets:
        - bucket-a
        - bucket-b
        - bucket-c
```

The `collector` field is the URL of your collector, for example `http://com.acme:8080/`. Under `s3` the `buckets` field is an array of the buckets you wish to monitor.
These buckets must exist prior to run and will not be created by the script.

The `app_id` field is the namespace to give your events, for example `com.snowplow.s3lambda`.

**N.B. bucket names here should not include the s3:// prefix**

### Running the deployment script

After you've edited the configuration file, you can deploy the AWS Lambda source and it's required permissions using:

```{bash}
python deploy.py
```

## Redshift

If you are running the Snowplow batch (Hadoop) flow with Amazon Redshift, then you should deploy the relevant event table into your Amazon Redshift.

You can find the table definition here:

[s3_notification_event_1.sql](https://github.com/snowplow/snowplow/blob/master/4-storage/redshift-storage/sql/com.amazon.aws.lambda/s3_notification_event_1.sql)

Make sure to deploy this table into the same schema as your `events` table.
