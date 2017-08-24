[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 1: setup a Collector**](Setting-up-a-collector) » [**Setup the Cloudfront collector**](Setting-up-the-Cloudfront-collector) » 1. Setup a bucket on S3 for the pixel

Log into your AWS account on [console.aws.amazon.com](https://console.aws.amazon.com) and select **S3** from the list of services offered. (Under **Storage & Content Delivery**.) You should be presented with a screen like the one below:

[[/setup-guide/images/cloudfront-collector-setup-guide/s3.png]]

We need to create a new bucket to store the 1x1 tracking pixel. To do this, simply click on the **Create Bucket** button on the top left of the screen:

[[/setup-guide/images/cloudfront-collector-setup-guide/s3-create-bucket.png]]

Enter a name for your bucket. Note every bucket name has to be *globally unique*.

Enter a bucket name and select a region. (The choice of region is not critical as the pixel will be served using Cloudfront. However, there are some privacy implications, especially for companies in the EU, that may mean you wish to select **Ireland** as your location: see [a note on privacy](#privacy) below).

**Do not** setup logging on this bucket. We will use Cloudfront, not S3, logging to record requests made for the tracking pixel.

Click the **Create** button on the bottom right of the popup. The new bucket should now be visible on the list of buckets on the left of the screen. On selecting it, you will get a warning that the bucket is empty. (We haven't added the tracking pixel yet!)

[[/setup-guide/images/cloudfront-collector-setup-guide/s3-bucket-created.png]]

<a name="privacy" />

## A note on privacy

Above we mentioned that, from a performance perspective, it is not important which Amazon data center you choose to self-host your pixel, or indeed your JavaScript:

[[/setup-guide/images/cloudfront-collector-setup-guide/02_choose_region.png]]

However, data center choice, particularly for your access logs, does matter from a data privacy perspective. For example, at the time of writing Amazon Web Services [recommends](http://aws.amazon.com/s3/faqs/#Can_I_comply_with_EU_data_privacy_regulations_using_Amazon_S3) storing data in the EU (Ireland) region if you wish to comply with EU data privacy regulations.

It is your responsibility to ensure that you comply with the privacy laws governing your web property and users.

## Alternative approach: AWS CLI

Alternatively, the above steps could be achieved with the following [AWS CLI](https://aws.amazon.com/cli/) commands.

To create a bucket in a specific region:

```sh
$ aws s3 mb s3://snowplow-static-demo --region us-west-1
```

The list of available regions could be obtained by executing:

```sh
$ aws ec2 describe-regions | grep 'RegionName'
```

You could also view it on the [Amazon doc page](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region).

If the bucket name is available you will get a confirmation on creation:

```sh
make_bucket: s3://snwplw-static-demo/
```

You can view the content of the bucket:

```sh
$ aws s3 ls s3://snwplw-static-demo
```

As no object has been uploaded yet, an empty list (shell prompt) is returned.

## All done?

Proceed to [step 2: upload the tracking pixel](2-upload-the-tracking-pixel).

Return to an [overview of the Cloudfront Collector setup](Setting-up-the-Cloudfront-collector).

Return to the [setup guide](setting-up-Snowplow).
