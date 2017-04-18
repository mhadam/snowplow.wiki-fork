[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [**Step 1: setup a Collector**](Setting-up-a-Collector) » [**Clojure collector setup**](setting-up-the-clojure-collector) » [[Download the Clojure collector WAR file or compile it from source]] » [[Create a new application in Elastic Beanstalk and upload the WAR file into it]] » [[Enable logging to S3]] » Configuring the Clojure collector

There are several settings to configure for your Clojure collector application:

1. [Enable SSH access to your Elastic Beanstalk environment and set your instances to have EBS backed root devices](#enable-ssh-access-to-your-elastic-beanstalk-environment-and-set-your-instances-to-have-ebs-backed-root-devices)
2. [Enable connection draining for your Elastic Beanstalk instance](#enable-connection-draining-for-your-elastic-beanstalk-instance)
3. [Configuring autoscaling settings](#configure-autoscaling-settings)

### Enable SSH access to your Elastic Beanstalk environment and set your instances to have EBS backed root devices

If you skipped enabling SSH access during initial [Clojure Collector setup](Create-a-new-application-in-Elastic-Beanstalk-and-upload-the-WAR-file-into-it), here's how you could do it now.

To start off, in the Elastic Beanstalk user interface, select your Clojure collector app, click on **Configuration** link and then on the cogwheel-like icon at the header of the **Instances** box.

[[/setup-guide/images/clojure-collector-setup-guide/21.jpg]]

#### Enable SSH access

This will enable you to SSH into one or more of you instances in the unlikely event that the rotation of collector logs from the bucket to S3 fails.

You should see a dropdown for selecting an EC2 key pair.

[[/setup-guide/images/clojure-collector-setup-guide/22.png]]

Select the key pair you want to be able to SSH into the instances with. Make sure that you keep the key you selected safe, so it is available should you need it.

### Use EBS-backed instances and set the HD size

Now we can update the instance type to use EBS root types. This means that in the unlikely event SSH access does not work, we can snapshot the contents of the instance and retrieve the logs from the snapshot.

From the **Root volume type** dropdown select **General Purpose (SSD)**

[[/setup-guide/images/clojure-collector-setup-guide/25.png]]

Now set the size of the instance HD. We recommend setting this to at least 100 GB. Remember that the events logged by the instance are stored locally to the instance and will then be flushed to S3 every hour - if you run out of HD space locally you will lose event data and the log rotation will fail, so it is essential to overprovision disk space.

When you are done, click the **Save** button. 

### Enable connection draining for your Elastic Beanstalk instance

Enabling this will enable us to make sure that in the event you want to scale down your cluster, you do not lose any log files generated on machines that will be terminated.

To do this, navigate to the EC2 section of the AWS console and select **Load Balancers** from the left-hand menu. Select the relevant Load Balancer from the list and click the **Edit** button in the **Connection Draining** section to enable this, as in the diagrams below:

[[/setup-guide/images/clojure-collector-setup-guide/23.png]]

[[/setup-guide/images/clojure-collector-setup-guide/24.png]]


### Configure autoscaling settings

Now that we've configured our cluster so that in the event of a problem we don't lose any data, we now need to set it up so it gracefully scales up to handle spikes in traffic.

In the Elastic Beanstalk interface, select your Clojure collector app, select **Configuration** and then **Scaling**

We recommend setting the following settings:

**Auto Scaling**

1. Minimum instance count: 2
2. Maximum instance count > 5

**Scaling Trigger**

1. Trigger measurement: CPU Utilization
2. Trigger statistic: Maximum
3. Unit of measurement: Percent
4. Measurement period: 5 minutes
5. Breach duration: 5 minutes
6. Upper threshold: < 60
7. Upper breach scale increment: 1
8. Lower threshold: 5
9. Lower breach scale increment: 0

Note that the above settings will cause your cluster to scale up to handle a spike in traffic but **not** to scale down following a drop. The reason is that automatically scaling down runs the risk that you will lose collector logs that have not been rotated to S3 prior to the instance being shut down. In the event that you need to scale a cluster down, we recommend that you do this manually, as described [here](Troubleshooting-Clojure-Collector-instances-to-prevent-data-loss).

Next: [[Enable support for HTTPS]]