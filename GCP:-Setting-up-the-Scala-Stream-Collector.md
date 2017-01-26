**This is pre-release**

## Contents

- 1. [Introduction](#intro)
- 2. [Enable and Setup Google Cloud Pub/Sub](#pubsub)
- 3. [Setting up the Scala Stream Collector](#ssc)
- 4. [Running the Collector](#running)
    * 4a. [locally (useful for testing)](#running-locally)
    * 4b. [on a GCP instance](#running-instance)
    * 4c. [on a load balanced auto-scaling GCP cluster](#running-cluster)
  

<a name="intro">
### 1. Introduction

The Scala Stream Collector can have Google Cloud Pub/Sub as its sink. Pub/Sub is a distributed Message Queue,
implemented completely on Google's infrastructure. Publisher applications publish to topics, whilst subscriber
applications listen to subscriptions, which can be set as pull or push, and with different acknowledgment
policies. For more on Pub/Sub go to: https://cloud.google.com/pubsub/docs/concepts

<a name="pubsub">
### 2. Enable and Setup Google Cloud Pub/Sub

- To use Pub/Sub, first we need to enable it. 
- Go to https://console.cloud.google.com/apis/api/pubsub.googleapis.com/overview
    * Make sure your project is selected (on the navbar, to the left of the search bar)
    * Click Enable
[[images/gcloud/gcloud-enable-pubsub.png]]

- You'll then have to create the topics to which the Scala Stream Collector publishes:
    * Click on the hamburger, on the top left corner
    * Type "Pub/Sub" or scroll down until you find it, under "Big Data"
    [[images/gcloud/gcloud-pubsub-sidebar.png]]
    * Create two topics: these will be the good topic and bad topic.
    [[images/gcloud/gcloud-pubsub-topics.png]]


<a name="ssc">
### 3. Setting up the Scala Stream Collector

- To set up the Scala Stream Collector, fill the apropriate fields of the config file
- Here's an example:

```
# 'collector' contains configuration options for the main Scala collector.
collector {
  # The collector runs as a web service specified on the following
  # interface and port.
  interface = "0.0.0.0"
  port = 8080 

  # Production mode disables additional services helpful for configuring and
  # initializing the collector, such as a path '/dump' to view all
  # records stored in the current stream.
  production = true

  # Configure the P3P policy header.
  p3p {
    policyref = "/w3c/p3p.xml"
    CP = "NOI DSP COR NID PSA OUR IND COM NAV STA"
  }

  # The collector returns a cookie to clients for user identification
  # with the following domain and expiration.
  cookie {
    enabled = false
    expiration =  "365 days"
    # Network cookie name
    name = sp
    # The domain is optional and will make the cookie accessible to other
    # applications on the domain. Comment out this line to tie cookies to
    # the collector's full domain
    domain = ""
  }

  # The collector has a configurable sink for storing data in
  # different formats for the enrichment process.
  sink {
    # Sinks currently supported are:
    # 'kinesis' for writing Thrift-serialized records to a Kinesis stream
    # 'kafka' for writing Thrift-serialized records to kafka
    # 'pubsub' for writing Thrift-serialized records to Google Cloud Pub/Sub
    # 'stdout' for writing Base64-encoded Thrift-serialized records to stdout
    #    Recommended settings for 'stdout' so each line printed to stdout
    #    is a serialized record are:
    #      1. Setting 'akka.loglevel = OFF' and 'akka.loggers = []'
    #         to disable all logging.
    #      2. Using 'sbt assembly' and 'java -jar ...' to disable
    #         sbt logging.
    enabled = "pubsub"

    kinesis {
      thread-pool-size: 10 # Thread pool size for Kinesis API requests

      # The following are used to authenticate for the Amazon Kinesis sink.
      #
      # If both are set to 'default', the default provider chain is used
      # (see http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html)
      #
      # If both are set to 'iam', use AWS IAM Roles to provision credentials.
      #
      # If both are set to 'env', use environment variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
      aws {
        access-key: "iam"
        secret-key: "iam"
      }

      # Data will be stored in the following stream.
      stream {
        region: "{{collectorSinkKinesisStreamRegion}}"
        good: "{{collectorKinesisStreamGoodName}}"
        bad: "{{collectorKinesisStreamBadName}}"
      }

      # Minimum and maximum backoff periods
      backoffPolicy: {
        minBackoff: 3000
        maxBackoff: 600000
      }
    }

    kafka {
      brokers: "{{collectorKafkaBrokers}}"

      # Data will be stored in the following topics
      topic {
        good: "{{collectorKafkaTopicGoodName}}"
        bad: "{{collectorKafkaTopicBadName}}"
      }
    }

    pubsub {
      topic {
        # these will be the names of the topics you created earlier
        good: "good"
        bad: "bad"
      }

      # if set to 'env' it will use the GOOGLE_APPLICATION_CREDENTIALS environment variable,
      # else, it assumes an absolute path
      google-auth-path: "env"
      google-project-id: "YOUR-PROJECT-ID"
    }

    # Incoming events are stored in a buffer before being sent to Kinesis/Kafka.
    # The buffer is emptied whenever:
    # - the number of stored records reaches record-limit or
    # - the combined size of the stored records reaches byte-limit or
    # - the time in milliseconds since the buffer was last emptied reaches time-limit
    buffer {
      byte-limit: 4000000
      record-limit: 500
      time-limit: 5000
    }
  }
}

# Akka has a variety of possible configuration options defined at
# http://doc.akka.io/docs/akka/2.2.3/general/configuration.html.
akka {
  loglevel = OFF  # 'OFF' for no logging, 'DEBUG' for all logging.
  loggers = ["akka.event.slf4j.Slf4jLogger"]
}

# spray-can is the server the Stream collector uses and has configurable
# options defined at
# https://github.com/spray/spray/blob/master/spray-can/src/main/resources/reference.conf
spray.can.server {
  # To obtain the hostname in the collector, the 'remote-address' header
  # should be set. By default, this is disabled, and enabling it
  # adds the 'Remote-Address' header to every request automatically.
  remote-address-header = on

  uri-parsing-mode = relaxed
  raw-request-uri-header = on

  # Define the maximum request length (the default is 2048)
  parsing {
    max-uri-length = 32768
  }
}

```


<a name="running" >
### 4. Running the Collector

- Download the Scala Stream Collector from Bintray. (The download destination will depend on where you want
to run the Collector)
- To run the collector, you'll need a config file as the one above.
- If the host where you're running is authenticated with an account authorized to use Pub/Sub, you won't need
the JSON credentials file and you should set "google-auth-path" to "env" in your config file. 
This is the case if you're running locally and have authenticated with
``` $ gcloud auth login ```
and also if you have set up the default Compute service account with Cloud Pub/Sub enabled (as we instructed above).
Otherwise :
    * You'll also need a credentials JSON file (whose path you'll have to specify
in the config file, as done in the example). 
    * For info on how to get the credentials JSON file 
refer to [this](https://github.com/snowplow/snowplow/wiki/GCP:-Getting-Started#auth)
    * Has described in that link, this JSON can only be obtained once, so it should be
handled carefully.
    * It should be **safely** placed wherever you intend to run it - don't e-mail it.

<a name="running-locally" >
#### 4a. locally (useful for testing)
To run the collector locally, assuming you have the above files in place,
simply run:
```
$ scala-stream-collector --config config.hocon
```

<a name="running-instance" >
#### 4b. on a GCP instance
To run the collector on a GCP instance, you'll first need to spin one up.
There are two ways to do so:

##### 4b-1. via dashboard
- Go to the [GCP dashboard](https://console.cloud.google.com/home/dashboard), and once again, make sure your project is selected.
- Click the hamburger on the top left corner, and select Compute Engine, under Compute
- Enable billing if you haven't (if you haven't enabled billing, at this point the only option you'll see is a button to do so)
[[images/gcloud/gcloud-instance-nobilling.png]]
- Click "Create instance" and pick the apropriate settings for your case, making sure of, at least the following:
    * Select Ubuntu LTS as the Boot disk
    * Under _Access scopes_, select "Set access for each API" and enable "Cloud Pub/Sub"
    * Under _Firewall_, select "Allow HTTP traffic"
    * Click _Management, disk, networking, SSH keys_
        - Under _Management_, add a Tag, such as "collector". (This is needed to add a Firewall rule)

[[images/gcloud/gcloud-instance-create1.png]]

[[images/gcloud/gcloud-instance-create2.png]]

- Click the hamburger on the top left corner, and click on "Networking", under _Compute_
- On the sidebar, click on "Firewall rules"
- Click "Create Firewall Rule"
- Name your rule
- Under _Source filter_ pick "Allow from any source"
- Under _Allowed protocols and ports_ add "tcp:8080"
    * Note that 8080 is the port assigned to the collector in the config file. If you choose another port here, make sure you change the config file
- Under _Target tags_ add the Tag with which you labeled your instance
- Click "Create"

[[/images/gcloud/gcloud-firewall.png]]

##### 4b-2. via command line
- Make sure you have authenticated as described above
- Here's an example command of an instance spin up: (check the [gcloud reference](https://developers.google.com/cloud/sdk/gcloud/reference/compute/?hl=en_US) for more info)

```

$ gcloud compute --project "example-project-156611" instances create "instance-2" \
                 --zone "us-central1-c" \
                 --machine-type "n1-standard-1" \
                 --subnet "default" \
                 --maintenance-policy "MIGRATE" \
                 --scopes 189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/pubsub",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/servicecontrol",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/service.management.readonly",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/logging.write",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/monitoring.write",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/trace.append",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/devstorage.read_only" \
                 --tags "collector" \
                 --image "/ubuntu-os-cloud/ubuntu-1604-xenial-v20170113" \
                 --boot-disk-size "10" \
                 --boot-disk-type "pd-standard" \
                 --boot-disk-device-name "instance-2"

$ gcloud compute --project "example-project-156611" firewall-rules create "default-allow-http" \
                 --allow tcp:80 \
                 --network "default" \
                 --source-ranges "0.0.0.0/0" \
                 --target-tags "collector"

$ gcloud compute --project "example-project-156611" firewall-rules create "collectors-allow-tcp8080" \
                 --allow tcp:8080 \
                 --network "default" \
                 --source-ranges "0.0.0.0/0" \
                 --target-tags "collector"

```


To place the above mentioned files in the instance (config file and the collector executable jar)
we suggest: for the jar, you'll wget it from Bintray into the instance directly; for the config file, store it using GCP Storage and then wget it into the instance. To store the config file:
- Click the hamburger on the top left corner and find Storage, under _Storage_
- Create a bucket
[[/images/gcloud/gcloud-storage1.png]]
- Then click "Upload Files" and upload your configuration file
- The link to your file will be https://storage.cloud.google.com/<YOUR-BUCKET-NAME>/<YOUR-CONFIG-FILE-NAME>


Once you have your config file in place, ssh into your instance:

```
$ gcloud compute ssh your-instance-name --zone your-instance-zone

```

And then run:
```
$ sudo apt-get update
$ sudo apt-get install default-jre
$ sudo apt-get install unzip
$ wget http://dl.bintray.com/snowplow/snowplow-generic/snowplow_scala_stream_collector_0.9.0.zip
$ wget https://storage.cloud.google.com/<YOUR-BUCKET-NAME>/<YOUR-CONFIG-FILE-NAME>
$ unzip snowplow_scala_stream_collector_0.9.0.zip
$ chmod +x snowplow-stream-collector-0.9.0
$ ./snowplow-stream-collector-0.9.0 --config <YOUR-CONFIG-FILE-NAME> &

```

<a name="running-cluster" >
#### 4c. a load balanced auto-scaling GCP cluster

To run a load balanced auto-scaling cluster, you'll need the following steps:

- Create an instance template
- Create an auto managed instance group
- Create a load balancer


##### Creating an instance template

- First you'll have to store your config file in some place that your instances can download it from.
- We suggest you store it in a GCP Storage bucket, as described above

###### via Google Cloud Console
- Click the hamburger on the top left corner and find "Compute Engine", under _Compute_
- Go to "Instance templates" on the sidebar. Click "Create instance template"
- Choose the appropriate settings for your case. Do (at least) the following:
    * Select Ubuntu LTS as the Boot disk
    * Under _Access scopes_, select "Set access for each API" and enable "Cloud Pub/Sub"
    * Under _Firewall_, select "Allow HTTP traffic"
    * Click _Management, disk, networking, SSH keys_

[[images/gcloud/gcloud-instance-template1.png]]

- Click "Management, disk, networking, SSH keys"
- Under _Management_, add a Tag, such as "collector". (This is needed to add a Firewall rule)
- Under "Startup script" add the following script (changing the relevant fields for your case):

**THIS HAS TO BE CHECKED**
```
#! /bin/bash
sudo apt-get update
sudo apt-get install default-jre
sudo apt-get install unzip
wget http://dl.bintray.com/snowplow/snowplow-generic/snowplow_scala_stream_collector_0.9.0.zip
wget https://storage.cloud.google.com/<YOUR-BUCKET-NAME>/<YOUR-CONFIG-FILE-NAME>
unzip snowplow_scala_stream_collector_0.9.0.zip
chmod +x snowplow-stream-collector-0.9.0
./snowplow-stream-collector-0.9.0 --config <YOUR-CONFIG-FILE-NAME> &
```
[[images/gcloud/gcloud-instance-template2.png]]

- Click "Create"
- Add a Firewall rule as described above (if you haven't already)

###### via command-line
Here's the command-line equivalent for the options selected by performing the steps above:

```

$ gcloud compute --project "example-project-156611" instance-templates create "ssc-instance-template" \
                 --machine-type "n1-standard-1" \
                 --network "default" \
                 --maintenance-policy "MIGRATE" \
                 --scopes 189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/pubsub",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/servicecontrol",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/service.management.readonly",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/logging.write",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/monitoring.write",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/trace.append",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/devstorage.read_only" \
                 --tags "collector" \
                 --image "/ubuntu-os-cloud/ubuntu-1604-xenial-v20170113" \
                 --boot-disk-size "10" \
                 --boot-disk-type "pd-standard" \
                 --boot-disk-device-name "ssc-instance-template" \
                 --metadata "startup-script=wget http://dl.bintray.com/snowplow/snowplow-generic/snowplow_scala_stream_collector_0.9.0.zip; wget http://link-to-your-config-file/config.hocon; unzip snowplow_scala_stream_collector_0.9.0.zip; chmod +x snowplow_scala_stream_collector; ./snowplow_scala_stream_collector --config config.hocon &"
```

##### Create an auto managed instance group

###### via Google Cloud Console

- On the side bar, click "Instance groups"
- Click "Create instance group"
- Fill in with the appropriate values. We named our instance group "collectors".
- Under _Instance template_ pick the instance template you created previously
- Set _Autoscaling_ to "On". By default the Autoscale is based on CPU usage and set with default settings, but you can change these to better suit your needs. We'll live them as they are for now.
- Under _Health Check_, pick "Create health check"
    * Name your health check
    * Under _Request path_ add "/health"
    * Click "Save and Continue"
- Click "Create"

[[images/gcloud/gcloud-group-create1.png]]
[[images/gcloud/gcloud-group-create2.png]]

###### via command-line
Here's the command-line equivalent for te options selected by performing the steps above:

```

$ gcloud compute --project "example-project-156611" instance-groups managed create "collectors" \
                 --zone "us-central1-c" \
                 --base-instance-name "collectors" \
                 --template "instance-template-1" \
                 --size "1" \

$ gcloud compute --project "example-project-156611" instance-groups managed set-autoscaling "collectors" 
                 --zone "us-central1-c" 
                 --cool-down-period "60" 
                 --max-num-replicas "10" 
                 --min-num-replicas "1" 
                 --target-cpu-utilization "0.6"

```


##### Configure the load balancer

- Click the hamburger on the top left corner, and find "Networking" under _Compute_
- On the side bar, click "Load Balancing"
- Click "Create load balancer", then click "Continue"
- Select "New TCP load balancer". Name your load balancer.

[[images/gcloud/gcloud-load-balancer1.png]]

- Under _Backend configuration_:
    * Pick the region where your instance group lives
    * Click "Select existing instance groups" and pick the instance group you created previously
    * Under _Health check_ pick the health check you created previously

[[images/gcloud/gcloud-load-balancer2.png]]

- Under _Frontend configuration_:
    * Leave _IP_ as "Ephemeral" and set _Port_ to 80
    * Add another one. Leave _IP_ as "Ephemeral" and set _Port_ to the port you added a Firewall rule for (8080 in the case of this example)

[[images/gcloud/gcloud-load-balancer3.png]]

- Click "Review and finalize" to check everything is OK.

- Click "Create"


