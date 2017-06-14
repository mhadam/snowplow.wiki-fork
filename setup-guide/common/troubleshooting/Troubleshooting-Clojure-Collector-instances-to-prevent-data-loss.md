The Clojure Collector is configured to upload logs of raw events to Amazon S3 every hour (typically at 10 minutes past the hour). Although this process is relatively robust, there are two failure scenarios which could lead to data loss:

1. Automatic log rotation fails, leaving logs building up on the collector instances
2. A collector instance is terminated, leading to the loss of the logs still contained on the instance (i.e. not yet uploaded to S3)

In this troubleshooting guide we will look at each failure scenario in turn.

### Prerequisites

To help prevent accidental data loss, please ensure you have setup your Elastic Beanstalk environment as follows:

#### SSH access to your Elastic Beanstalk environment is enabled

This involves specifying an SSH keypair when starting up the environment. Without this, you won't be able to log in to your instance(s) as required later.

#### Connection draining is enabled for your Elastic Beanstalk environment

This can be found under Configuration section for the environment within the Load Balancer section. This will be useful to make sure that you don't sever any existing connections while swapping over instances. You can find more information about this feature here:

http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/TerminologyandKeyConcepts.html#conn-drain

#### Instances should be Elastic Block Store-backed at root

This should (untested) allow you to safely bounce Clojure Collector instances without losing the logs.

#### Automatic downscaling should be disabled

In the Elastic Beanstalk UI, under Scaling and then Scaling Trigger, make sure that the Lower breach scale increment is set to 0, not -1 or similar. 

### Failure scenario 1: automatic log rotation fails

The solution is to log into the instance and manually force the push of the remaining logs. If you ssh into the instance, you can run the following three commands to capture the current logs and upload them to S3:

```
sudo logrotate -f /etc/logrotate.elasticbeanstalk.hourly/logrotate.elasticbeanstalk.tomcat8.conf
sudo logrotate -f /etc/logrotate.elasticbeanstalk.hourly/logrotate.elasticbeanstalk.httpd.conf
sudo publishLogs.py --de-dupe --conf-path '/opt/elasticbeanstalk/tasks/publishlogs.d/*' --location-prefix resources/environments/logs/publish/ --num-concurrent 2
```

Note that this process is normally performed by cron and you can find the relevant entries under `/etc/cron.hourly` and `/etc/cron.d` if you're interested in taking a look.

Make sure that you check that the manually-pushed logs make it to S3 and are collected correctly.

### Failure scenario 2: a collector instance is terminated

There are three scenarios in which a collector instance can be terminated:

1. An instance has a hardware or software failure and gets replaced by AutoScaling
2. AutoScaling spins up multiple instances for a load spike and they scale back down during a less busy time
3. You or one of your team manually terminate an instance (e.g. because it has become unresponsive)

There is currently no workaround for scenario 1. We avoid scenario 2 by configuring our Elastic Beanstalk environment to never automatically scale down.

For scenario 3, the case where you need to manually terminate a Clojure Collector instance, the AWS team have compiled a set of steps which should prevent data loss. For simplicity, the following guide assumes that you have a single instance environment and wish to replace that instance with a new instance (terminating the old one) without either:

1. Losing any logs of raw events contained on the Clojure Collector instance and not uploaded to S3
2. Dropping any events being sent by the trackers (i.e. switchover has no downtime)

Let's get started:

#### Step 1. Change the AutoScaling termination policy

In the EC2 section of the AWS Management Console, under AutoScaling Groups, find the AutoScaling group for your environment (`awseb-e-xxx-stack-AWSEBAutoScalingGroup-xxx`). In the details section, select edit and change Termination Policies to OldestInstance only, removing Default. The reason for this and the next step is due to the way AutoScaling works. If there are an equal number of instances in Availability Zones, then an AZ would first be selected at random, then the relevant termination policy applied. In your case, if we want to remove the oldest instance when we scale down, we need to ensure they are both in the same AZ so that it can evaluate the two instances correctly. You can find more information on how it all works here:

http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/us-termination-policy.html#your-termination-policy

#### Step 2. Reduce the number of Availability Zones for your environment to 1

The reason for this is due to the way that AutoScaling termination works as explained in the prior step. Do this under the configuration/scaling section and set the scaling to Any 1 AZ and select the AZ to where your instance currently resides.

#### Step 3. Increase your minimum instance count to 2

Once done, you would then need to monitor the environment and wait for the instance to be available before proceeding and you should see when it's ready in the recent events under the environment dashboard. Please make sure that you wait for the new instance to be up and running and in the environment, so that there is no loss of traffic.

#### Step 4. De-register the instance you want to remove from the ELB manually

Within the EC2 management console, under Load Balancers, find the ELB for your environment (`awseb-e-b-xxx-xxx`) and manually remove your target instance (i-ecxxx) from the ELB within the instances section. With connection draining turned on, incoming connections will as much as possible gracefully moved over to the other instance before the instance is removed. The instance will still be part of the environment and AutoScaling group and remain available, but it will not be responding to requests because it has been removed. This means that the logs should be in a consistent state at this point.

#### Step 5. Decrease your minimum instance count to 1

Now that you have the logs in S3, this will terminate the instance. As the termination policy is for the AutoScaling group is to remove the Oldest Instance, it will then terminate the correct instance.

#### Step 6. Change the AutoScaling termination policy back to Default

Return to your default settings for AutoScaling, as it's the most cost-effective under normal operation. Just reverse the change you make to the termination policy earlier. You'd have to wait for the environment to stabilize before performing this action.

These instructions should leave you with a new running instance and no break in recording while you are performing the changeover.
