<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 4: setting up alternative data stores**](Setting-up-alternative-data-stores) > Setting up Amazon DynamoDB

## Contents

1. [Setting up IAM Policy](#policy)
2. [Creating Amazon DynamoDB table](#table)
3. [Next steps](#next-steps)

**Note**: We recommend running all Snowplow AWS operations through an IAM user with the bare minimum permissions required to run Snowplow. Please see our [IAM user setup page](IAM-setup) for more information on doing this.

<a name="policy" />

## 1. Creating Amazon DynamoDB table

If [[Relational Database Shredder]] won't find a specified table - it will try to create it with default provisioned throughput, which might be not sufficient. This step is optional, but recommended.

Table name can be anything, but must be unique.

Partition key must be called `eventId` and have type String.
Sort key must be called `fingerprint` and have type String.

Uncheck "Use default settings" checkbox and set "Write capacity units" to 100.
Capacity units value is individual and should tweaked depending on your cluster size.

[[/setup-guide/images/dynamodb-setup-guide/create-table.png]]

After table is created, write down "Amazon Resource Name (ARN)" in "Overview" tab. It should look similar to `arn:aws:dynamodb:us-east-1:719197435995:table/one-more-deduplication-test` This ARN will be used in next step.

[[/setup-guide/images/dynamodb-setup-guide/table-arn.png]]

<a name="policy" />

## 2. Setting up IAM Policy

Log into the AWS console, navigate to the IAM section and go to **Policies**:

Select **Create Your Own Policy** and choose descriptive name. Paste following JSON as Policy Document:

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
                "arn:aws:dynamodb:us-east-1:719197435995:table/snowplow-deduplication"
            ]
        }
    ]
}
```

Notice element in `Resourse` array. It must be changed to your ARN from previous step.

`dynamodb:CreateTable` and `dynamodb:DeleteTable` are unnecessary if you already created table.

[[/setup-guide/images/dynamodb-setup-guide/policy.png]]

Back to [top](#top).

<a name="next-steps" />

## 3. Next steps

Now you have setup DynamoDB, you are ready to run [[Relational Database Shredder]] job with cross-batch de-duplication.
