[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 1: setup a Collector**](Setting-up-a-Collector) > [**Clojure collector setup**](setting-up-the-clojure-collector) > [[Download the Clojure collector WAR file or compile it from source]] » Create a new application in Elastic Beanstalk and upload the WAR file into it

Amazon makes it easy to create a new application in Elastic Beanstalk and upload your `war` file into it. All of this is possible via the web UI.

On your web browser, log into the [AWS control panel][aws]. From the **Services** dropdown menu select **Elastic Beanstalk**. 

Before you create your application, you need to switch to the region you want your web server located. Select your region from the dropdown on the top right of the screen:

[[/setup-guide/images/clojure-collector-setup-guide/0.jpg]]

Once you've selected your region, you're ready to create your application. Click on the **Create New Application** button (located on the top right of the screen, below the top menu bar with the region selection). Give your application a suitable name and description.

[[/setup-guide/images/clojure-collector-setup-guide/1.jpg]]

Next, you will need to set up the appropriate environment for the application to run in. Hit **Create web server** button.

[[/setup-guide/images/clojure-collector-setup-guide/2.jpg]]

For the platform, select **Tomcat**. In this demo, we picked a "*Single instance*". However, you could set up a load balanced, auto scaling environment.

[[/setup-guide/images/clojure-collector-setup-guide/3.jpg]]

For the **Application Source**, select **Upload your own**. Use the **Choose File** button to point Elastic Beanstalk at the `war` file from [part 1](Download-the-Clojure-collector-WAR-file-or-compile-it-from-source). Click **Next** to upload the file.

[[/setup-guide/images/clojure-collector-setup-guide/4.jpg]]

Once uploaded, give your environment a suitable name, URL and description. Ensure the environment URL is available (click **Check availability** button).

[[/setup-guide/images/clojure-collector-setup-guide/5.jpg]]

The following screen should be left unmodified. We do **not** need a database to run the collector. Nor do we require it running in VPC. 

[[/setup-guide/images/clojure-collector-setup-guide/6.jpg]]

Next, we need to specify another set of configuration details. Set a suitable instance type (we recommend at least `m1.small`). If you have an EC2 key pair configured, you can enter the key pair name at this stage: this will enable you to use the key pair to SSH in should you wish. (This is not required, and can be added later without any difficulty.)

[[/setup-guide/images/clojure-collector-setup-guide/7.jpg]]

Also at this stage, you could set the instance to use EBS root types (*Root Volume* section). This means that in the unlikely event SSH access does not work, we can snapshot the contents of the instance and retrieve the logs from the snapshot.

From the *Root volume type* dropdown select **General Purpose (SSD)**. Then set the size of the instance HD. We recommend setting this to at least 100 GB. Remember that the events logged by the instance are stored locally to the instance and will then be flushed to S3 every hour - if you run out of HD space locally you will lose event data and the log rotation will fail, so it is essential to overprovision disk space.

If you skip either selecting EC2 key pair or configuring Root Volume you could set it up later on as shown on the [Configuring the Clojure collector](Configuring-the-Clojure-collector) page.

[Tagging the environment](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.tagging.html) is optional and more appropriate if you run multiple environments and you want to distinguish them in cost allocation reports.

[[/setup-guide/images/clojure-collector-setup-guide/8.jpg]]

Give the appropriate permissions and click **Next** to review the submitted information so far.

[[/setup-guide/images/clojure-collector-setup-guide/9.jpg]]

Once you are satisfied with with the summary hit **Launch** button at the very bottom of the ***Review*** page (not shown on the screenshot). 

[[/setup-guide/images/clojure-collector-setup-guide/10.jpg]]

Elastic Beanstalk will start creating your environment followed by launching the application. The whole process could take a while.

[[/setup-guide/images/clojure-collector-setup-guide/11.jpg]]

Once the application is up and running the dashboard will present the *Recent Events* logs and hopefully report the successful launch.

[[/setup-guide/images/clojure-collector-setup-guide/12.jpg]]

To test that all is working as expected, click on the **URL** link at the very top of the Dashboard. (This is `http://cc-endpoint-demo.us-west-2.elasticbeanstalk.com` in the example above.) This should return a 404. If you add `/i` to the path (e.g. `http://cc-endpoint-demo.us-west-2.elasticbeanstalk.com/i`), right click on the window and select **Inspect Element** in Chrome or **Inspect with Firebug** in Firefox, you should be able to see if the cookie has been set:

[[/setup-guide/images/clojure-collector-setup-guide/8.png]]


## Alternative approach: AWS CLI

Alternatively, the above steps could be done via [AWS CLI](https://aws.amazon.com/cli/) in the following manner.

1.Create an empty application.

```sh
$ aws elasticbeanstalk create-application --application-name "Clojure Collector Demo" --description "Enables cross domain user tracking"
```

2.Set Clojure Collector WAR as an application.

```sh
$ aws elasticbeanstalk create-application-version \
    --application-name "Clojure Collector Demo" \
    --version-label cc-endpoint-demo \
    --cli-input-json "{\"SourceBundle\":{\"S3Bucket\":\"snowplow-hosted-assets\",\"S3Key\":\"2-collectors/clojure-collector/clojure-collector-1.1.0-standalone.war\"}}"
```

3.Create environment making sure you provide the valid `solution-stack-name`. Note the `environment-name` will form the collector endpoint URL.

```sh
$ aws elasticbeanstalk create-environment \
    --application-name "Clojure Collector Demo" \
    --environment-name cc-endpoint-demo \
    --solution-stack-name "64bit Amazon Linux 2015.09 v2.0.4 running Tomcat 8 Java 8" \
    --version-label cc-endpoint-demo \
    | jq -r '.EnvironmentId'
```

***Hint***: The list of available solutions could be obtained with the command `aws elasticbeanstalk list-available-solution-stacks  | jq -r '.SolutionStacks[]' | grep "Tomcat"`.

Launching environment can take several minutes. On output you will get `EnvironmentId`. We will refer to it as `{{ CC_ENV }}` in the next step where we'll show you how to enable S3 logging.

Next: [[enable logging to S3]]

[aws]: https://console.aws.amazon.com/
