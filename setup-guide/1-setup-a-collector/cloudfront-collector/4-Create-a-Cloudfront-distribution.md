[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 1: setup a Collector**](Setting-up-a-collector) » [**Setup the Cloudfront collector**](Setting-up-the-Cloudfront-collector) » [**1. Setup a bucket on S3 for the pixel**](1-Setup-a-bucket-on-S3-for-the-pixel) » [**2. Upload the Tracking Pixel**](2-upload-the-tracking-pixel) » [**3. Create a bucket for the Cloudfront logs**](3-Create-a-bucket-for-Cloudfront-logs) » 4. Create a Cloudfront distribution

Having set up everything in S3, we now need to create a Cloudfront distribution. This will be used to serve the tracking pixel `i` (so we need to tell Cloudfront to serve the contents of the [first bucket](1-Setup-a-bucket-on-S3-for-the-pixel) in S3 we created, that houses the tracking pixel). We also need to switch on Cloudfront logging, so that every request made for the tracking pixel by the Snowplow tracker will be logged. Again, we need to tell Cloudfront to store these logs in the [second bucket](3-create-a-bucket-for-cloudfront-logs) we created.

## 4.1 Switch from S3 to Cloudfront

Click on the **Services** menu on the top left of the browser screen, and select **Cloudfront** from the drop down:

[[/setup-guide/images/cloudfront-collector-setup-guide/switch-to-cloudfront.jpg]]

You should see a screen like the following:

[[/setup-guide/images/cloudfront-collector-setup-guide/cloudfront-create-distribution-1.jpg]]

## 4.2 Create a new Cloudfront distribution / subdomain

Click on the **Create Distribution** button on the top of the window (shown in the previous screenshot). When presented with the following screen, click the **Get Started** button under the   **Web** (rather than **RTMP**) distribution section.

[[/setup-guide/images/cloudfront-collector-setup-guide/cloudfront-create-distribution-2.jpg]]

Now we are presented with a screen with many options:

[[/setup-guide/images/cloudfront-collector-setup-guide/cloudfront-create-distribution-3.jpg]]

The first field, **Origin Domain Name** lets us specify where Cloudfront should find the content to distribute on the Cloudfront subdomain we are in the process of creating. If you click on it, you will be presented with a drop down:

[[/setup-guide/images/cloudfront-collector-setup-guide/cloudfront-create-distribution-4.jpg]]

In the drop down you should see the bucket you setup in [step 1](1-Setup-a-bucket-on-S3-for-the-pixel) that contains the tracking pixel `i`. Selet this - the **Origin ID** field should be automatically populated for you:

[[/setup-guide/images/cloudfront-collector-setup-guide/cloudfront-create-distribution-5.jpg]]

Great! We've linked the Cloudfront subdomain to the bucket on S3 with our tracking pixel. Now we need to switch on Cloudfront logging and ensure that Cloudfront logs to the bucket we setup in S3. Scroll down the list of options (you will need to scroll quite far):

[[/setup-guide/images/cloudfront-collector-setup-guide/cloudfront-create-distribution-6.jpg]]

Change the radio button for **Logging** from **Off** to **On**. The two fields beneath it should be activated. Now click on the **Buckets for logging** field:

[[/setup-guide/images/cloudfront-collector-setup-guide/cloudfront-create-distribution-7.jpg]]

You should now be able to select the [2nd bucket](3-create-a-bucket-for-cloudfront-logs) you setup to store the logs.

Now all you need to do is tell Cloudfront to create the distribution. Scroll down to the end of the options and select the **Create Distribution** button:

[[/setup-guide/images/cloudfront-collector-setup-guide/cloudfront-create-distribution-8.jpg]]

You should see your new distribution listed:

[[/setup-guide/images/cloudfront-collector-setup-guide/cloudfront-create-distribution-9.jpg]]

Note it might take a while for the distribution creation to complete. Watch for the **Status** changed from "*In Progress*" to "*Deployed*". 

Write down the **Domain Name** for the distribution you just created. (Highlighted above - in our case it is `http://d3age8pcob9fi8.cloudfront.net`.) You will need this in the next step (to test the collector is working), and when you set up your [tracker](choosing-a-tracker).

## Alternative approach: AWS CLI

Note [AWS CLI](https://aws.amazon.com/cli/) support for **cloudfront** service is only available in a *preview stage*.

However, you will be able to execute the commands to list the current distributions and query their status.

List current distribution:

```sh
$ aws cloudfront list-distributions
```

Wait for distribution deployment to complete:

```sh
aws cloudfront wait distribution-deployed --id {{ DISTRIBUTION_ID }}
```

The last command requires the distribution ID. As a deployment process takes a while to complete the command will poll every 60 seconds until a successful state has been reached. This will exit with a return code of 255 after 25 failed checks.

## All done?

Proceed to [step 5: test your pixel](5-Test-your-pixel).

Return to an [overview of the Cloudfront Collector setup](Setting-up-the-Cloudfront-collector).

Return to the [setup guide](setting-up-Snowplow).