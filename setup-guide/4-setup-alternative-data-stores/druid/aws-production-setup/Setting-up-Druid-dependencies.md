<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 4: setting up alternative data stores**](Setting-up-alternative-data-stores) > [**Setup Druid*](Setting-up-Druid) > [**Setup Druid for production in AWS**](Setting-up-Druid-for-production) > Setup Druid dependencies

The prerequisites for a production setup of Druid in AWS are as follows:

1. [Amazon S3][amazon-s3] to act as the data repository for Druid ("deep storage")
2. [Postgres on Amazon RDS][pg-rds] to act as the metadata storage for Druid
3. [Apache ZooKeeper][zookeeper] to coordinate the Druid clusters

Let's configure/install each of these in turn.

<a name="prereq-s3" />

### 1. Amazon S3

We will use [Amazon S3][amazon-s3] as the data repository for Druid ("deep storage").

ADD REST OF SECTION

<a name="prereq-postgres-rds" />

### 2. Postgres on Amazon RDS

We will use a PostgreSQL instance running on Amazon RDS as the medata storage for Druid.

ADD REST OF SECTION

<a name="prereq-zookeeper" />

### 3. Apache ZooKeeper

We will use Apache ZooKeeper as the cluster coordination service for Druid.

**Setting up and running a production ZooKeeper cluster is out of the scope of this documentation. We strongly recommend reading [ZooKeeper (O'Reilly)][zookeeper-oreilly] before proceeding.**

Create a ZooKeeper cluster on EC2, with an **odd** number of nodes, at least 3. You should **not** attempt to run any other Druid components on these ZooKeeper nodes.

[amazon-s3]: https://aws.amazon.com/s3/
[pg-rds]: https://aws.amazon.com/rds/postgresql/
[apache-zookeeper]: https://zookeeper.apache.org/
[zookeeper-oreilly]: http://shop.oreilly.com/product/0636920028901.do
