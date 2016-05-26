[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 2c: Setup a Data Store Source**](Setting-up-a-Data-Store-Source) » **AWS Lambda source setup**

##Overview

This tool builds into a single zip file which can be uploaded as a new [AWS Lambda](http://docs.aws.amazon.com/lambda/latest/dg/welcome.html) function. The description you give your Lambda will be used as your collector endpoint (an URL). This means the Lambda you create will need the ability to perform `lambda:GetFunctionConfiguration` on itself, amongst the other normal AWS lambda permissions.

## Deployment guide

This deployment assumes you have the AWS-cli[link], and Python ~2.7 installed.

```{bash}
sudo pip install pyyaml
wget <download_bundle>
unzip <download_bundle>
cd <download_bundle>
```

At this point, fire up your favourite editor and edit the file `config.yaml` (see below)

### Editing `deploy/config.yaml`

This configures your deployment. The configuration file is written in [YAML](yaml-link) and looks as below:

```{yaml}
snowplow:
    collector: http://a-collector.com
s3:
    buckets:
        - bucket-a
        - bucket-b
        - bucket-c
```

The `collector` field is the URL of your collector, for example `http://com.acme:8080/`. Under `s3` the `buckets` field is an array of the buckets you wish to monitor. 
These buckets must exist prior to run and will not be created by the script. 

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