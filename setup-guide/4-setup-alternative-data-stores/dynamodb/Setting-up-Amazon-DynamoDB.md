<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 4: setting up alternative data stores**](Setting-up-alternative-data-stores) > Setting up Amazon DynamoDB

## Contents

1. [Setting up IAM Policy](#policy)
2. [Creating Amazon DynamoDB table](#table)
3. [Next steps](#next-steps)

**Note**: We recommend running all Snowplow AWS operations through an IAM user with the bare minimum permissions required to run Snowplow. Please see our [IAM user setup page](IAM-setup) for more information on doing this.

<a name="policy" />
## 1. Creating Amazon DynamoDB table

If [[Scala Hadoop Shred]] won't find a specified table - it will try to create it with default provisioned throughput, which might be not enough. This step is optional, but recommended.

Table name can be anything, but must be unique.

Partition key must be called `eventId` and have type String.
Sort key must be called `fingerprint` and have type String.

Uncheck "Use default settings" checkbox and set "Write capacity units" to 100.

**TODO** specify provisioned throughput

**TODO** image

After table is created, notice "Amazon Resource Name (ARN)" in "Overview" tab. It should look similar to `arn:aws:dynamodb:us-east-1:719197435995:table/one-more-deduplication-test` This ARN will be used in next step.

<a name="policy" />
## 2. Setting up IAM Policy

Log into the AWS console, navigate to the IAM section and go to **Policies**:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1486765706000",
            "Effect": "Allow",
            "Action": [
                "dynamodb:CreateTable",
                "dynamodb:DeleteTable",
                "dynamodb:DescribeTable",
                "dynamodb:PutItem"
            ],
            "Resource": [
                "arn:aws:dynamodb:us-east-1:719197435995:table/one-more-deduplication-test"
            ]
        }
    ]
}
```

Notice element in `Resourse` array. It must be changed to your ARN from previous step.

`dynamodb:CreateTable` and `dynamodb:DeleteTable` are unnecessary if you already created table.


**TODO** image

[[/setup-guide/images/postgresql/aws-ec2-console.jpg]]

Back to [top](#top).

<a name="next-steps" />
## 4. Next steps

Now you have setup DynamoDB, you are ready to run [[Scala Hadoop Shred]] job with cross-batch de-duplication.

[amazon-emr-guide]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html
[setup-storageloader]: 1-Installing-the-StorageLoader
[postgresql-table-def]: https://github.com/snowplow/snowplow/blob/master/4-storage/postgres-storage/sql/atomic-def.sql
[simon-rumble]: https://github.com/shermozle
