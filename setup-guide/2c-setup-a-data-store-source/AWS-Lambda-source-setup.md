[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 2c: Setup a Data Store Source**](Setting-up-a-Data-Store-Source) » **AWS Lambda source setup**

##Overview

This tool builds into a single zip file which can be uploaded as a new [AWS Lambda](http://docs.aws.amazon.com/lambda/latest/dg/welcome.html) function. The description you give your Lambda will be used as your collector endpoint (an URL). This means the Lambda you create will need the ability to perform `lambda:GetFunctionConfiguration` on itself, amongst the other normal AWS lambda permissions.

##Using the `deploy.py` script

It's recommended you launch this script from within Vagrant, as the dependencies are pre built. The deploy script works using a configuration file `deploy/config.yaml`, this must be edited
to suit prior to running the script.

To get started, clone this repository:

```
git clone git@github.com:snowplow/snowplow-aws-lambda-source.git
```

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

Once you have edited the configuration, you can execute `deploy.py` to deploy and configure the S3 source on AWS. It's strongly recommended you use Vagrant, however providing
you have the `aws-cli` tools installed and configured, a python 2.7 install (including pyyaml) then you can run the script without Vagrant.

```
# Assuming you're in the root of the cloned directory
vagrant up && vagrant ssh
cd deploy
python deploy.py
```

## More information on uploading a AWS Lambda function manually is available [here](http://docs.aws.amazon.com/lambda/latest/dg/with-s3.html).

## Redshift

If you are running the Snowplow batch (Hadoop) flow with Amazon Redshift, then you should deploy the relevant event table into your Amazon Redshift.

You can find the table definition here:

[s3_notification_event_1.sql](https://github.com/snowplow/snowplow/blob/master/4-storage/redshift-storage/sql/com.amazon.aws.lambda/s3_notification_event_1.sql)

Make sure to deploy this table into the same schema as your `events` table.