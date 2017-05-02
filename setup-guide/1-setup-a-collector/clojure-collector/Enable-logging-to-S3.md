[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 1: setup a Collector**](Setting-up-a-Collector) » [**Clojure collector setup**](setting-up-the-clojure-collector) » [[Download the Clojure collector WAR file or compile it from source]] » [[Create a new application in Elastic Beanstalk and upload the WAR file into it]] » Enable logging to S3

Now that your application is up and running, you need to update the configuration so that the Tomcat logs are pushed to S3. (These will be processed by the ETL step to generate your Snowplow data.) Click on the **Configuration** link.

[[/setup-guide/images/clojure-collector-setup-guide/13.jpg]]

Within the ***Web Tier*** section locate *Software Configuration* box and click on the cogwheel-like icon to navigate to the configuration page.

[[/setup-guide/images/clojure-collector-setup-guide/14.jpg]]

Scroll down the page to the *Logs Option* section and check the box marked **Enable log file rotation to Amazon S3**:

[[/setup-guide/images/clojure-collector-setup-guide/15.jpg]]

Click the **Apply** button. The environment update will be processed, and the collector app should be live again after a few minutes.

[[/setup-guide/images/clojure-collector-setup-guide/16.jpg]]


## Alternative approach: AWS CLI

The above effect could be achieved via [AWS CLI](https://aws.amazon.com/cli/) in the following fashion.

```sh
$ aws elasticbeanstalk update-environment \
    --environment-id {{ CC_ENV }} \
    --option-settings Namespace="aws:elasticbeanstalk:hostmanager",OptionName="LogPublicationControl",Value="true"
```

Note the value for `environment-id` is obtained from the [previous step](Create-a-new-application-in-Elastic-Beanstalk-and-upload-the-WAR-file-into-it).

## Checking that your logs are being pushed to S3

Amazon pushes Tomcat access logs to S3 hourly. (In our experience, they generally appear 10 minutes passed each hour.) Finding your logs in S3, however, is not entirely trivial...

On the Amazon Web Services console, navigate to the EC2 section of the site, and click on the **Instances** item in the Navigation menu on the left hand side. A list of all the EC2 instances you are running should show as below:

[[/setup-guide/images/clojure-collector-setup-guide/17.jpg]]

You should see at least one instance with the name given to the environment you created above. (In our case, 'cc-endpoint-demo'.) Select it:

[[/setup-guide/images/clojure-collector-setup-guide/18.jpg]]

There are two identifiers we need to take note of. The first is the **security group** (highlighted in the screenshot above). Here we are interested in the characters between `awseb-` and `-stack-AWSEBSecurityGroup` i.e. in our case, `e-uqdkjdyrax`. The other identifier we need to note is the Instance identifier listed both in the summary (under the instances list) and in the more detailed pane. In our case, this is `i-81902a46`.

Now that we have those two identifiers, we can locate the logs in S3. In the web console, navigate to the S3 section. On the left hand side, you should see at least one bucket with a name that begins `elasticbeanstalk` and then follows with the region you setup Elastic Beanstalk in:

[[/setup-guide/images/clojure-collector-setup-guide/19.jpg]]

Select that bucket - you should find a folder within it called `resources`. Within that folder you should find another called `environments`. Within that folder you should find a logs folder, within that a publish folder and within that a folder with the security group identifier you noted above. (In our case `e-uqdkjdyrax`.) Within that folder you should find another with your instance identifier, also noted above. (In our case, `i-81902a46`.) The path to your logs is therefore:

	s3://elasticbeanstalk-{{REGION NAME}}-{{UUID}}/resources/environments/logs/publish/{{SECURITY GROUP IDENTIFIER}}/{{INSTANCE IDENTIFIER}}

[[/setup-guide/images/clojure-collector-setup-guide/20.jpg]]

Write this path down somewhere as you will need it later to configure [EmrEtlRunner](setting-up-EmrEtlRunner).

**Important**: do not include an {{INSTANCE IDENTIFIER}} at the end of your path. This is because your Clojure Collector may end up logging into multiple {{INSTANCE IDENTIFIER}} folders (if set up to run in *auto scaling* as opposed to *single instance* mode). That is Elastic Beanstalk might spin up more instances to run the Clojure collector to cope with a spike in traffic. By specifying your In Bucket only to the level of the Security Group identifier, you make sure that Snowplow can process all logs from all instances as the EmrEtlRunner processes all logs in all subfolders.

Note - if you cannot find the logs as described above, it may be because Amazon has not written any yet. You may need to wait an hour for the logs to appear.

Next: [configure the Clojure collector](Configuring-the-Clojure-collector)

