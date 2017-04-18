[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 1: setup a Collector**](Setting-up-a-collector) » [**Setup the Cloudfront collector**](Setting-up-the-Cloudfront-collector) » [**1. Setup a bucket on S3 for the pixel**](1-Setup-a-bucket-on-S3-for-the-pixel) » [**2. Upload the Tracking Pixel**](2-upload-the-tracking-pixel) » 3. Create a bucket for the Cloudfront logs

Click on the **Create bucket** button again, this time to create a second bucket for storing the log files that Cloudfront will generate. Give the bucket a suitable name.

[[/setup-guide/images/cloudfront-collector-setup-guide/s3-create-logs-bucket.jpg]]

## Alternative approach: AWS CLI

Alternatively, the above step could be achieved with the following [AWS CLI](https://aws.amazon.com/cli/) command.

To create a bucket in a specific region:

```sh
$ aws s3 mb s3://snowplow-logs-demo --region us-west-1
```

The list of available regions could be obtained by executing:

```sh
$ aws ec2 describe-regions | grep 'RegionName'
``` 

You could also view it on the [Amazon doc page](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region).

If the bucket name is available you will get a confirmation on creation:

```sh
make_bucket: s3://snwplw-logs-demo/
```

You can view the content of the bucket:

```sh
$ aws s3 ls s3://snwplw-logs-demo
```

As no object has been uploaded yet, an empty list (shell prompt) is returned.

## All done?

Proceed to [step 4: create a Cloudfront distribution](4-create-a-Cloudfront-distribution).

Return to an [overview of the Cloudfront Collector setup](Setting-up-the-Cloudfront-collector).

Return to the [setup guide](setting-up-Snowplow).