<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 4: setting up alternative data stores**](Setting-up-alternative-data-stores) > Setting up Amazon DynamoDB

## Contents

1. [Setting up IAM Policy](#policy)
2. [Creating Amazon DynamoDB table](#table)
3. [Next steps](#next-steps)

**Note**: We recommend running all Snowplow AWS operations through an IAM user with the bare minimum permissions required to run Snowplow. Please see our [IAM user setup page](IAM-setup) for more information on doing this.

<a name="policy" />
## 1. Setting up IAM Policy

Log into the AWS console, navigate to the IAM section and go to **Policies**:

TODO policy

TODO image
[[/setup-guide/images/postgresql/aws-ec2-console.jpg]]

<a name="policy" />
## 2. Creating Amazon DynamoDB table

This step is optional, but recommended. If [[Scala Hadoop Shred]] won't find a specified table - it will try to create it with default provisioned throughput.


TODO image

Back to [top](#top).

<a name="next-steps" />
## 4. Next steps

Now you have setup PostgreSQL, you are ready to [setup the StorageLoader][setup-storageloader] to automate the regular loading of Snowplow data into the PostgreSQL events table.

[amazon-emr-guide]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html
[setup-storageloader]: 1-Installing-the-StorageLoader
[postgresql-table-def]: https://github.com/snowplow/snowplow/blob/master/4-storage/postgres-storage/sql/atomic-def.sql
[simon-rumble]: https://github.com/shermozle
