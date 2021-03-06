<a name="top" />

## Overview

Setting up permissions in IAM for the user(s) installing Snowplow is an 3 step process:

- 1 [Create an IAM group (incl. creating a user and setting permissions)](#create-group)
  - 1.1 [Batch Permissions](#batch-perms)
  - 1.2 [Kinesis Permissions](#kinesis-perms)
  - 1.3 [Kinesis & Batch](#combined-perms)
- 2 [Enable users to log into AWS](#enable-login)


**Disclaimer: Snowplow Analytics Ltd will not be liable for any problems caused by the full or partial implementation of these instructions on your Amazon Web Services account. If in doubt, please consult an independent AWS security expert.**

_Warning: these permissions are still more permissive than they need to be. We will be putting in time to narrow them down further over the coming weeks._

<a name="create-group" />

## 1. Setup the IAM group

### Initial group configuration

First click on the IAM icon on the AWS dashboard:

[[/setup-guide/images/iam/console-iam.png]]

Now click on the _Create a New Group of Users_ button:

[[/setup-guide/images/iam/new-iam-group.png]]

### Group Name

Enter a _Group Name_ of `snowplow-setup`:

[[/setup-guide/images/iam/new-iam-group-snowplow.png]]

### Permissions

Now choose the _Custom Policy_ option and click _Select_:

[[/setup-guide/images/iam/new-iam-group-custom-policy.png]]

Let's give it a _Policy Name_ of `snowplow-policy-setup-infrastructure`:

[[/setup-guide/images/iam/new-iam-group-policy-name.png]]

### Permissions Outline:

Depending on what Pipeline needs to be setup you will need slightly varying permissions.

<a name="batch-perms" />

#### Batch Permissions

The following permissions are needed for all batch proccessing operations:

* Amazon S3
* Amazon EMR
* Amazon EC2 (required for EmrEtlRunner)
* Amazon Marketplaces (required for EmrEtlRunner)
* Amazon CloudFront (required for Cloudfront collector)
* Amazon Elastic Beanstalk (required for Clojure collector)
* Amazon Elasticsearch service (for loading bad rows)
* Amazon Redshift (required for Redshift)
* Amazon Cloudformation (required if the Snowplow team setup your Snowplow data pipeline, as we use Cloudformation)
* Amazon IAM (required as part of the Clojure collector setup, as a role is created for the Clojure collector application)
* Amazon RDS (PostgreSQL server required by Iglu Server)

**If you are not using the Clojure Collector, you can remove the Elastic Beanstalk section.**

Paste the following JSON into the _Policy Document_ text area:

```javascript
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "acm:*",
        "autoscaling:*",
        "aws-marketplace:ViewSubscriptions",
        "aws-marketplace:Subscribe",
        "aws-marketplace:Unsubscribe",
        "cloudformation:*",
        "cloudfront:*",
        "cloudwatch:*",
        "ec2:*",
        "elasticbeanstalk:*",
        "elasticloadbalancing:*",
        "elasticmapreduce:*",
        "es:*",
        "iam:*",
        "rds:*",
        "redshift:*",
        "s3:*",
        "sns:*"
      ],
      "Resource": "*"
    }
  ]
}
```

<a name="kinesis-perms" />

#### Kinesis Permissions

For the Kinesis Pipeline ending in an Elasticsearch Cluster you will need these permissions:

* Amazon EC2
* Amazon Cloudformation (required if the Snowplow team setup your Snowplow data pipeline, as we use Cloudformation)
* Amazon IAM (required as each Application for the Kinesis Pipeline has its own IAM Policy)
* Amazon Kinesis (required for Kinesis Streams to be created)
* Amazon DynamoDB (required for Kinesis Applications to work with Kinesis Streams)
* Amazon CloudWatch/Logs
* Amazon AutoScaling (required for all Kinesis Applications to work effectively)
* Amazon ElasticLoadBalancing (required for the Kinesis Collector)

Paste the following JSON into the _Policy Document_ text area:

```javascript
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "acm:DescribeCertificate",
        "acm:ListCertificate",
        "autoscaling:*",
        "elasticloadbalancing:*",
        "kinesis:*",
        "iam:*",
        "cloudwatch:*",
        "ec2:*",
        "cloudformation:*",
        "cloudfront:*",
        "logs:*",
        "dynamodb:*",
        "sns:*"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```

<a name="combined-perms" />

#### Kinesis & Batch Permissions

For the Kinesis Pipeline ending in the LZO S3 Sink with events from this sink then being processed in batches you will need these permissions:

* Amazon S3
* Amazon EMR
* Amazon EC2
* Amazon Marketplaces (required for EmrEtlRunner)
* Amazon Redshift (required for Redshift)
* Amazon Cloudformation (required if the Snowplow team setup your Snowplow data pipeline, as we use Cloudformation)
* Amazon IAM (required as each Application for the Kinesis Pipeline has its own IAM Policy)
* Amazon RDS (PostgreSQL server required by Iglu Server)
* Amazon Kinesis (required for Kinesis Streams to be created)
* Amazon DynamoDB (required for Kinesis Applications to work with Kinesis Streams)
* Amazon CloudWatch/Logs
* Amazon AutoScaling (required for all Kinesis Applications to work effectively)
* Amazon ElasticLoadBalancing (required for the Kinesis Collector)

Paste the following JSON into the _Policy Document_ text area:

```javascript
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "acm:*",
        "autoscaling:*",
        "aws-marketplace:Subscribe",
        "aws-marketplace:Unsubscribe",
        "aws-marketplace:ViewSubscriptions",
        "cloudformation:*",
        "cloudfront:*",
        "cloudwatch:*",
        "dynamodb:*",
        "ec2:*",
        "es:*",
        "elasticbeanstalk:*",
        "elasticloadbalancing:*",
        "elasticmapreduce:*",
        "iam:*",
        "kinesis:*",
        "logs:*",
        "rds:*",
        "redshift:*",
        "s3:*",
        "sns:*"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```

### After Policy Selection

Once you have selected the correct _Policy Document_ click _Continue_:

[[/setup-guide/images/iam/new-iam-group-policy-continue.png]]

### Users

From the _Add Existing Users_ tab, switch to the _Create New Users_ tab:

[[/setup-guide/images/iam/new-iam-group-create-new-user.png]]

Now enter a first _User Name_ - we use `snowplow-setup`:

[[/setup-guide/images/iam/new-iam-group-create-new-user-snowplow.png]]

Keep the option _Generate an access key for each User_ checked, and then click _Continue_.

### Review

Check that the configuration for your new IAM group looks something like this:

[[/setup-guide/images/iam/new-iam-group-review.png]]

Click _Continue_ and you should see the following:

[[/setup-guide/images/iam/new-iam-group-download-credentials.png]]

Click _Download Credentials_ to save these credentials locally. Then click _Close Window_.

Provide these credentials in a secure way - **not** via email - to whoever is setting up Snowplow for you, so that they can add them into the configuration of your EmrEtlRunner applications.

Back to [top](#top).

<a name="enable-login" />

## 2. Allow the IAM user to login

For much of the Snowplow setup process, the IAM user you have setup above will need access to the Amazon Web Services control panel.

From within the _Users_ tab inside the IAM dashboard, click on your `snowplow` user:

[[/setup-guide/images/iam/user-snowplow.png]]

Now switch to the _Security Credentials_ tab in the bottom pane, and click _Manage Password_ on the right:

[[/setup-guide/images/iam/user-snowplow-manage-password.png]]

Now choose _Assign an auto-generated password_:

[[/setup-guide/images/iam/user-snowplow-auto-password.png]]

Click _Apply_ and you should see the following:

[[/setup-guide/images/iam/user-snowplow-download-credentials.png]]

Click _Download Credentials_ to save these credentials locally. Then click _Close Window_.

Now, provide the following details in a secure way - **not** via email - to whoever is setting up Snowplow for you:

* Login URL: [https://snplow.signin.aws.amazon.com/console](https://snplow.signin.aws.amazon.com/console)
* Username: `snowplow`
* Password: as downloaded

Back to [top](#top).

[iam]: http://aws.amazon.com/iam/
[operate-snowplow]: Setup-IAM-permissions-for-operating-Snowplow
