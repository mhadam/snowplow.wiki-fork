<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 4: setting up alternative data stores**](Setting-up-alternative-data-stores) > Setup Redshift

Setting up Redshift is a 9 step process:

1. [Launch a Redshift cluster](#launch)
2. [Authorize client connections to your cluster](#authorise)  
3. [Connect to your cluster](#connect)
4. [Set up the Snowplow database and events table](#db)
5. [Set up user access on Redshift](#user)
6. [Generate Redshift-format data from Snowplow](#etl)
7. [Update the search path for your Redshift cluster](#search_path)
8. [Automate the loading of Snowplow data into Redshift](#load)

**Note**: We recommend running all Snowplow AWS operations through an IAM user with the bare minimum permissions required to run Snowplow. Please see our [IAM user setup page](IAM-setup) for more information on doing this.

<a name="launch" />
## 1. Launch a Redshift Cluster

Go into the Amazon webservices console and select "Redshift" from the list of services.

[[/setup-guide/images/redshift-setup-guide/1.png]]

Click on the "Launch Cluster" button:

[[/setup-guide/images/redshift-setup-guide/2.png]]

Enter suitable values for the cluster identifier, database name (e.g. 'snowplow'), port, username and password. Click the "Continue" button.

[[/setup-guide/images/redshift-setup-guide/3.png]]

We now need to configure the cluster size. Select the values that are most appropriate to your situation. We generally recommend starting with a single node cluster with node type i.e. a `dw1.xlarge` or `dw2.large` node, and then adding nodes as your data volumes grow.

You now have the opportunity to encrypt the database and and set the availability zone if you wish. Select your preferences and click "Continue".

[[/setup-guide/images/redshift-setup-guide/4.png]]

Amazon summarises your cluster information. Click "Launch Cluster" to fire your Redshift instance up. This will take a few minutes to complete.

Alternatively, you could use [AWS CLI](https://aws.amazon.com/cli/) to launch a new cluster. The outcome of the above steps could be achieved with the following command.

```sh
$ aws redshift create-cluster \
    --node-type dc1.large \
    --cluster-type single-node \
    --cluster-identifier snowplow \
    --db-name pbz \
    --master-username admin \
    --master-user-password TopSecret1
```

<a name="authorise" />
## 2. Authorize client connections to your cluster

You authorize access to Redshift differently depending on whether the client you're authorizing is an EC2 instance or not

2.1 [EC2 instance](#ec2)  
2.2 [Other client](#other)

<a name="ec2" />
### 2.1 Granting access to an EC2 instance

TO WRITE

<a name="other" />
### 2.2 Granting access to non-EC2 boxes

To enable a direct connection between a client (e.g. on your local machine) and Redshift, click on the cluster you want to access, via the AWS console:

[[/setup-guide/images/redshift-setup-guide/5.png]]

Click on "Security Groups" on the left hand menu.

[[/setup-guide/images/redshift-setup-guide/6.png]]

Amazon lets you create several different security groups so you can manage access by different groups of people. In this tutorial, we will just update the default group to grant access to a particular IP address.

Select the default security group:

[[/setup-guide/images/redshift-setup-guide/7.png]]

We need to enable a connection type for this security group. Amazon offers two choices: an 'EC2 Security Group' (if you want to grant access to a client running on EC2) or a CIDR/IP connection if you want to connect a clieint that is not an EC2 instance.

In this example we're going to establish a direct connection between Redshift and our local machine (not on EC2), so select CIDR/IP. Amazon helpfully guesses the CIDR of the current machine. In our case, this is right, so we enter the value:

[[/setup-guide/images/redshift-setup-guide/8.png]]

and click "Add".

We should now be able to connect a SQL client on our local machine to Amazon Redshift.

**Note:** Amazon has moved to launching Redshift clusters in a VPC instance by default. In this case, the process for adding IP addresses or EC2 instances to a security group is very similar, but rather than being done in the `Redshift > Security Groups` section of the AWS console, it is done in the `EC2 -> VPC security groups` section of the AWS management console.

Via [AWS CLI](https://aws.amazon.com/cli/), you could create the security group in the following fashion.

```sh
$ aws ec2 create-security-group \
    --group-name "Redshift unlimited access" \
    --description "Unsafe. Use for demonstration only" \
    --vpc-id {{ VPC_ID }} \
    | jq -r '.GroupId'
```

On output you'll get `GroupId`. We'll refer to it as `{{ REDSHIFT_SG }}`.

Next, you need to add access rules to the new security group (amend as required to serve your purpose).

```sh
$ aws ec2 authorize-security-group-ingress \
    --group-id {{ REDSHIFT_SG }} \
    --protocol tcp \
    --port 5439 \
    --cidr 0.0.0.0/0
```

Then tie the previously created security to the cluster in the following manner. On output you'll get the cluster address which you can use in place of `hostname` when establishing the connection to your database.

```sh
$ aws redshift modify-cluster \
    --cluster-id snowplow \
    --vpc-security-group-ids {{ REDSHIFT_SG }} \
    | jq -r '.Cluster.Endpoint.Address'
```

<a name="connect" />
## 3. Connect to your cluster

There are two ways to connect to your Redshift cluster:

4.1 [Directly](#directly)  
4.2 [Via SSL](#ssl)  

<a name="directly" />
### 3.1 Directly connect

Amazon has helpfully provided detailed instructions for connecting to Redshift using [SQL Workbench] [sql-workbench-tutorial]. In this tutorial we will connect using [Navicat](http://www.navicat.com/), a database querying tool which we recommend (30 day trial versios are available from the [Navicat website](http://www.navicat.com/)).

**Note: Redshift can be accessed using PostgreSQL JDBC or ODBC drivers. Only specific versions of these drivers work with Redshift**. These are:

* JDBC [[http://jdbc.postgresql.org/download/postgresql-8.4-703.jdbc4.jar]]
* ODBC [[http://ftp.postgresql.org/pub/odbc/versions/msi/psqlodbc_08_04_0200.zip]] or [[http://ftp.postgresql.org/pub/odbc/versions/msi/psqlodbc_09_00_0101-x64.zip]] for 64 bit machines

We've found that many other Postgres clients generally do work with Redshift for *most* queries. (Especially where those queries are being hand crafted rather than generated by the client itself, so that the user can make sure any SQL is Redshift-compatible.)

Open Navicat, select "Connection" -> "PostgreSQL" to establish a new connection:

[[/setup-guide/images/redshift-setup-guide/9.png]]

Give your connection a suitable name. We now need to enter the host name, port, database, username and password. With the exception of password, these are all available directly from the AWS UI. Go back to your browser, open the AWS console, go to Redshift and select your cluster:

[[/setup-guide/images/redshift-setup-guide/10.png]]

Copy the endpoint, port, database name and username into the relevant fields in Navicat, along with the password you created when you setup the cluster:

[[/setup-guide/images/redshift-setup-guide/11.png]]

Click "Test Connection" to check that it is working. Assuming it is, click "OK".

[[/setup-guide/images/redshift-setup-guide/12.png]]

The Redshift cluster is now visible on Navicat, alongside every other database it is connected to.

<a name="ssl" />
### 3.2 Connect via SSL

Amazon Redshift supports Secure Sockets Layer (SSL) connections to encrypt data and server certificates to validate the server certificate that the client connects to. 

ODBC DSNs contain an `sslmode` setting that determines how to handle encryption for client connections and server certificate verification. The following `sslmode` values from the client connection are supported:

- `disable`: SSL is disabled and the connection is not encrypted.
- `allow`: SSL is used if the server requires it.
- `prefer`: SSL is used if the server supports it. Amazon Redshift supports SSL, so SSL is used when you set `sslmode` to `prefer`.
- `require`: SSL is required.
- `verify-ca`: SSL must be used and the server certificate must be verified.
- `verify-full`: SSL must be used. The server certificate must be verified and the server hostname must match the hostname attribute on the certificate.

The difference between `verify-ca` and `verify-full` depends on the policy of the root CA. If a public CA is used, `verify-ca` allows connections to a server that somebody else may have registered with the CA to succeed. In this case, `verify-full` should always be used. If a local CA is used, or even a self-signed certificate, using verify-ca often provides enough protection.

The default value for `sslmode` is `prefer`. It is only provided as the default for backwards compatibility, and not recommended in secure deployments. 

<a name="db" />
## 4. Set up the Snowplow database and events table

Now that you have Redshift up and running, you need to create the Snowplow database (if you didn't do this as part of the process of firing up your Redshift cluster) and creating your Snowplow events table.

To create a new database on Redshift, right click on the new connection and select 'New database'. Give your database a suitable name and click OK.

The Snowplow events table definition for Redshift is available on the repo [here] [redshift-table-def]. Execute the queries in the file - this can be done using psql as follows:

Navigate to your snowplow github repo:

	$ cd snowplow

Navigate to the sql file:

	$ cd 4-storage/redshift-storage/sql

Now execute the `atomic-def.sql` file:

	$ psql -h <HOSTNAME> -U {{ admin_username }} -d snowplow -p <PORT> -f atomic-def.sql

Where `{{ admin_username }}` is the username you created when you setup the Redshift cluster.

If you prefer using a GUI (e.g. Navicat) rather than `psql`, you can do so. These will let you either run the files directly, or you can simply copy and paste the queries in the files into your GUI of choice, and execute them from there.

If you capture unstructured events or contexts, you also need to create the corresponding tables in Redshift. For example:

	$ psql -h <HOSTNAME> -U {{ admin_username }} -d snowplow -p <PORT> -f com.snowplowanalytics.snowplow/mobile_context_1.sql
	$ psql -h <HOSTNAME> -U {{ admin_username }} -d snowplow -p <PORT> -f com.snowplowanalytics.snowplow/link_click_1.sql
	$ psql -h <HOSTNAME> -U {{ admin_username }} -d snowplow -p <PORT> -f org.w3/performance_timing_1.sql

<a name="user" />
## 5. Set up user access on Redshift

We recommend you setup access credentials for at least three different users:

1. [The StorageLoader](#storageloader-user)
2. [A user with read-only access](#read-only-user)  to the data, but write access on his / her own schema
3. [A power user](#power-user) with admin privilages

<a name="storageloader-user" />
### 5.1 Creating a user for the StorageLoader

We recommend that you create a specific user in Redshift with *only* the permissions required to load data into your Snowplow schema and run `vacuum` and `analyze` against those tables, and use this user's credentials in the StorageLoader config to manage the automatic movement of data into the table. That way, in the event that the server running StorageLoader is hacked and the hacker gets access to those credentials, they cannot use them to do any harm to your other data in Redshift. To create a new user with restrictive permissions, log into Redshift, connect to the Snowplow database and execute the following SQL:

```sql
CREATE USER storageloader PASSWORD '$storageloaderpassword';
GRANT USAGE ON SCHEMA atomic TO storageloader;
GRANT INSERT ON TABLE "atomic"."events" TO storageloader;
```

You can set the user name and password (`storageloader` and `$storageloaderpassword` in the example above) to your own values. Note them down: you will need them when you come to setup the storageLoader in the next phase of the your Snowplow setup.

It's important that both `vacuum` and `analyze` are run on a regular basis. These can only be run by a superuser or the owner of the table. The latter is the more restricted solution, so we transfer ownership on all tables in atomic to the StoreLoader user. This can be done by running the following query against all tables in atomic:

```sql
ALTER TABLE atomic.events OWNER TO storageloader;
```

<a name="read-only-user" />
### 5.2 Creating a read-only user

To create a new user who can read Snowplow data, but not modify it, connect to the Snowplow database and execute the following SQL:

```sql
CREATE USER read_only PASSWORD '$read_only_user';
GRANT USAGE ON SCHEMA atomic TO read_only;
GRANT SELECT ON TABLE atomic.events TO read_only;
```

The last query would need to be run for each table in atomic.

Lastly, we may want to let create a schema in Redshift where the read-only user can create his/ her own tables for analytics purposes, for example:

```sql
CREATE SCHEMA scratchpad;
GRANT ALL ON SCHEMA scratchpad TO read_only;
```

<a name="power-user" />
### 5.3 Creating a power user

To create a power user that has super user privilages, connect to the Snowplow database in Redshift and execute the following:

```sql
create user power_user createuser password '$poweruserpassword';
GRANT ALL ON DATABASE snowplow TO power_user;
GRANT ALL ON SCHEMA atomic TO power_user;
GRANT ALL ON TABLE atomic.events TO power_user;
```

Note that now you've created your different users, we recommend that you no longer use the credentials you created when you created the Redshift cluster originally.

<a name="etl" />
## 6. Generate Redshift-format data from Snowplow

Assuming you are working through the setup guide sequentially, you will have already  ([setup EmrEtlRunner] [emr-etl-runner]). You should therefore have Snowplow events in S3, ready for uploading into Redshift.

If you have not already [setup EmrEtlRunner] [emr-etl-runner], then please do so now, before proceeding onto the [next stage](#load).

<a name="search_path" />
## 7. Update the search path for your Redshift cluster

The `search path` specifies where Redshift should look to locate tables and views that are specified in queries submitted to it. This is important, because the Snowplow events table is located in the "atomic" schema, whilst derived tables can be located in their own schemas. By adding these schemas to the Redshift search path, it means that when you connect to Redshift from different tools (e.g. Tableau, SQL workbench), those tools can identify tables and views in each of those schemas, and present them as options for the user to connect to.

Updating the search path is straightforward. In the AWS Redshift console, click on the **Parameters Group** menu item on the left hand. menu, and select the button to **Create Cluster Parameter Group**:

[[/setup-guide/images/redshift-setup-guide/13.png]]

Give your parameter group a suitable name and click **Create**. The parameter group should appear in your list of options.

Now open up your parameter group, by clicking on the magnifying glass icon next to it, and then selecting **Edit** in the menu across the top:

[[/setup-guide/images/redshift-setup-guide/14.png]]

Update the **search_path** section to read the following:

	atomic,  derived

Note: you can choose to add and remove schemas. Do note, however, that if you include a schema on the search path that does not exist yet on your database, you will cause Redshift to become very unstable. (For that reason, it is often a good idea to leave the `search_path` with the default settings, and only update it once you've setup the relevant schemas in Redshift.)

Save the changes. We now need to update our cluster to use this parameter group. To do so, select **Clusters** from the left hand manu, select your cluster and click the modify button. Now you can select your new parameter group in the **Cluster Parameter Group** dropdown:

[[/setup-guide/images/redshift-setup-guide/15.png]]

Click the **Modify** button to save the changes. We now need to reboot the cluster, so that the new settings are applied. Do this by clicking the **Reboot** button on the top menu.


<a name="load" />
## 8. Loading Snowplow data into Redshift using StorageLoader

Now that you have your Snowplow database and table setup on Redshift, you are ready to setup the StorageLoader to regularly upload Snowplow data into the table.

1. [Installing the StorageLoader](1-Installing-the-StorageLoader)
2. [Using the StorageLoader](2-using-the-storageloader)
3. [Scheduling the StorageLoader](3-scheduling-the-storageloader)

[Back to top](#top).


[emr-etl-runner]: 1-Installing-EmrEtlRunner
[storage-loader]: 1-Installing-the-StorageLoader
[sql-workbench-tutorial]: http://docs.aws.amazon.com/redshift/latest/gsg/getting-started.html
[redshift-table-def]: https://github.com/snowplow/snowplow/blob/master/4-storage/redshift-storage/sql/atomic-def.sql
