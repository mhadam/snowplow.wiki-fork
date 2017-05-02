<a name="top" />

## Overview

These are instructions for setting up the IAM permissions for the "user(s)" that "operates" Snowplow: in practice this is the user associated with the credentials that should be used in the EmrEtlRunner and StorageLoader config files. The permissions represent the minimum required to keep the Snowplow data pipeline running: this is best practice, so that if a hacker manages to compromise the server with EmrEtlRunner and StorageLoader on it (so gain access to these credentials), they will have only limited access to your AWS resources.

Setting up the credentials is an 5 step process:

- 1 [Create a new IAM group](#create-group)
- 2 [Set the permissions for the group](#permissions)
    - [Running Kinesis Applications](#kinesis-apps)
- 3 [Create a new user](#user)
- 4 [Add the new user to your new group](#add-to-group)
- 5 [Update the EmrEtlRunner and StorageLoader config files with the new credentials](#update-configs)
- 6 [Delete the user created to setup Snowplow](#delete)

**Disclaimer: Snowplow Analytics Ltd will not be liable for any problems caused by the full or partial implementation of these instructions on your Amazon Web Services account. If in doubt, please consult an independent AWS security expert.**

<a name="create-group" />

## 1. Setup the IAM group

### Initial group configuration

First click on the IAM icon on the AWS dashboard:

[[/setup-guide/images/iam/console-iam.png]]

Now click on the _Create a New Group of Users_ button:

[[/setup-guide/images/iam/new-iam-group.png]]

### Group Name

Enter a _Group Name_ of `snowplow-data-pipeline`:

[[/setup-guide/images/iam/operating-snowplow-permissions/group-name-snowplow-data-pipeline.png]]

Then click continue.

Back to [top](#top).

<a name="permissions" />

## 2. Set the permissions for the group

Choose the *Custom Policy* option and click *Select*:

[[/setup-guide/images/iam/new-iam-group-custom-policy.png]]

Let's give it a _Policy Name_ of `snowplow-policy-operate-datapipeline`.

We now need to create the Amazon policy document to define *just* the user permissions required to run the Snowplow pipeline. An example permissions policy is given below:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ec2:*",
                "elasticbeanstalk:*",
                "elasticmapreduce:*",
                "s3:*",
                "sns:*",
                "iam:passrole",
                "cloudformation:ListStackResources",
                "cloudformation:DescribeStacks",
                "autoscaling:DescribeAutoScalingGroups",
                "redshift:DescribeClusters"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
```

Note that there should be opportunities to lock these permissions down further e.g.:

1. Place them on specific resources rather than `*`
2. Remove e.g. `sns` permissions if you are not using SNS to monitor your Snowplow infrastructure

Copy and paste the JSON into the 

[[/setup-guide/images/iam/operating-snowplow-permissions/permissions-entered.png]]

Click *Continue*:

[[/setup-guide/images/iam/operating-snowplow-permissions/add-existing-users.png]]

When given the opportunity, do not add an existing user. We want to create a new user with these permissions, who **only** has these permissions.

Review the final settings before pressing *Continue* to complete the process. Your new group is now setup.

Back to [top](#top).

<a name="kinesis-apps" />

### 2.1 Running Kinesis Applications

Our Kinesis Applications are designed so that you can launch them all with IAM Roles so you will never have to store your AWS Access/Secret Keys on the instance itself. Lock down your permissions as far as possible, and remember that assigning permissions to specific instances of autoscaling groups is better than assigning them to users.

Back to [top](#top).

<a name="user" />

## 3. Create a new user

Now that our group has been created, we need to add a new user to it.

In the IAM console, click on the *Users* section on the left hand menu:

[[/setup-guide/images/iam/operating-snowplow-permissions/users-section.png]]

Click on the *Create New Users* button:

[[/setup-guide/images/iam/operating-snowplow-permissions/create-new-user.png]]

Give your new user a suitable name e.g. `snowplow-operator`. Click *Create*:

[[/setup-guide/images/iam/operating-snowplow-permissions/download-credentials.png]]

AWS gives you the chance to either show or download the credentials. Whichever you do, make sure you **store these credentials safely**. You will need them in [step 5](#update-configs) of this guide.

Now close the window: your new user is setup.

Back to [top](top).

<a name="add-to-group" />

## 4. Add the new user to your new group

The user we have created has no permissions -> we need to add her to the new group we created to give her those permissions.

To do that, click on the *Groups* section on the AWS console, and select the new group you created:

[[/setup-guide/images/iam/operating-snowplow-permissions/select-group.png]]

Click on the *Add Users to Group* button:

[[/setup-guide/images/iam/operating-snowplow-permissions/add-user-to-group.png]]

The user now has the required permissions.

Back to [top](top).

<a name="update-configs" />

## 5. Update the EmrEtlRunner and StorageLoader config files with the new credentials

Now that you have setup your new user and given her the relevant permissions to run the Snowplow data pipeline, you need to take those credentials and use them instead of the existing credentials in your EmrEtlRunner and StorageLoader config files.

Those files should be accessible on the server setup to run EmrEtlRunner and StorageLoader. (Examples of those files can be found on the Snowplow repo [here][emretlrunner.config] and [here][storageloader.config].) Update the `:access_key_id:` and `:secret_access_key:` fields with those from the new user in **both** files. 

Back to [top](#top).

<a name="delete" />

## 6. Delete the user created to setup Snowplow

Now that we have created a new user with just the permissions required to run the Snowplow data pipeline, and used her credentials in in the EmrEtlRunner and StorageLoader config files, we can delete the user that we created to setup/install Snowplow originally.

In the IAM console, go into the `snowplow-setup` group you created when you created user credentials for the individual who setup Snowplow. Select the user in that group e.g. `snowplow-setup` and click the *Remove User from Group* link:

[[/setup-guide/images/iam/operating-snowplow-permissions/remove-snowplow-setup.png]]

Click *Remove from Group* when AWS asks you to confirm.

[[/setup-guide/images/iam/operating-snowplow-permissions/confirm-remove.png]]

In the event that you need to update your Snowplow setup in the future, you can simply create a new user, fetch their credentials, then add them to the `setup-snowplow` group to give them the relevant permissions.

Back to [top](#top).



[emretlrunner.config]: https://github.com/snowplow/snowplow/tree/master/3-enrich/emr-etl-runner/config
[storageloader.config]: https://github.com/snowplow/snowplow/tree/master/4-storage/storage-loader/config