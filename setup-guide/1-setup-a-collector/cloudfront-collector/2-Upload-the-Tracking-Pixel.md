[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 1: setup a Collector**](Setting-up-a-collector) » [**Setup the Cloudfront collector**](Setting-up-the-Cloudfront-collector) » 2. Upload the tracking pixel

You can download a copy of the tracking pixel from the [Snowplow Github repo](https://github.com/snowplow/snowplow/tree/master/2-collectors/cloudfront-collector/static). One convenient way to quickly grab `i` is to execute the following at the command line:

```sh
$ wget https://github.com/snowplow/snowplow/raw/master/2-collectors/cloudfront-collector/static/i
```

To upload the tracking pixel into the bucket you just created, click on the **Upload** button on the top left corner of the window. A popup will appear:

[[/setup-guide/images/cloudfront-collector-setup-guide/upload-i-to-s3.jpg]]

Click on the **+ Add Files** button and select the tracking pixel from the location you downloaded it to on your local machine. (Alternative instructions are provided within the popup window). The tracking pixel file should be listed above the **+ Add Files** button. When it is, click the **Start Upload** button on the bottom right of the popup.

When the upload is complete, the pixel should be listed in the bucket:

[[/setup-guide/images/cloudfront-collector-setup-guide/upload-i-to-s3-complete.jpg]]

Now we need to make the file public so that it is accessible to anyone visiting your website(s) or mobile app. Right click on the file and select **Make Public** from the menu:

[[/setup-guide/images/cloudfront-collector-setup-guide/i-make-public.jpg]]

Confirm that you want to make the file public and Amazon should complete the operation:

[[/setup-guide/images/cloudfront-collector-setup-guide/i-make-public-complete.jpg]]

As a final step, we will set the mimetype on the tracking pixel to `image/gif` - otherwise the pixel is transmitted as an `application/octet-stream`, which causes browser warnings.

So, click on the tracking pixel, click on the Properties tab, scroll down to the Metadata section and then click Add more metadata and add an entry for **Content-Type** like so:

[[/setup-guide/images/cloudfront-collector-setup-guide/i-set-image-mimetype.jpg]]

Hit Save and then you should be done.

## Alternative approach: AWS CLI

Alternatively, the above steps could be achieved with the following [AWS CLI](https://aws.amazon.com/cli/) commands.

Download the tracking pixel to the local machine:

```sh
$ wget https://github.com/snowplow/snowplow/raw/master/2-collectors/cloudfront-collector/static/i
```

Upload it to the bucket and set the appropriate permissions and MIME type:

```sh
$ aws s3 cp i s3://snwplw-static-demo --acl public-read --content-type image/gif
```

It will be confirmed with the following output

```sh
upload: ./i to s3://snwplw-static-demo/i
```

## All done?

Proceed to [step 3: create a bucket for Cloudfront logs](3-create-a-bucket-for-cloudfront-logs).

Return to an [overview of the Cloudfront Collector setup](Setting-up-the-Cloudfront-collector).

Return to the [setup guide](setting-up-Snowplow).