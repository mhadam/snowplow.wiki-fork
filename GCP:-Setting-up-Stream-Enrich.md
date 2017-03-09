**This is pre-release**

## Contents

- 1. [Introduction](#intro)
- 2. [Enable and Setup Google Cloud Pub/Sub](#pubsub)
- 3. [Setting up Stream Enrich](#se)
- 4. [Running Stream Enrich](#running)
    * 4a. [locally (useful for testing)](#running-locally)
    * 4b. [on a GCP instance](#running-instance)
   
<a name="intro">
### 1. Introduction

Stream Enrich can now Google Cloud Pub/Sub as its source and sink. Pub/Sub is a distributed Message Queue,
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

- You'll then have to create the topics to which Stream Enrich publishes and subscribes:
    * Click on the hamburger, on the top left corner
    * Type "Pub/Sub" or scroll down until you find it, under "Big Data"
    [[images/gcloud/gcloud-pubsub-sidebar.png]]
    * Create two topics: these will be the good topic and bad topic.
    [[images/gcloud/gcloud-pubsub-topics.png]]


<a name="se">
### 3. Setting up Stream Enrich

- To set up the Scala Stream Collector, fill the apropriate fields of the config file
- Here's an example:

```
# Copyright (c) 2013-2016 Snowplow Analytics Ltd. All rights reserved.
#
# This program is licensed to you under the Apache License Version 2.0, and
# you may not use this file except in compliance with the Apache License
# Version 2.0.  You may obtain a copy of the Apache License Version 2.0 at
# http://www.apache.org/licenses/LICENSE-2.0.
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the Apache License Version 2.0 is distributed on an "AS
# IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied.  See the Apache License Version 2.0 for the specific language
# governing permissions and limitations there under.

# This file (application.conf.example) contains a template with
# configuration options for Stream Enrich.

enrich {
  # Sources currently supported are:
  # 'kinesis' for reading Thrift-serialized records from a Kinesis stream
  # 'kafka' for reading Thrift-serialized records from a Kafka topic
  # 'stdin' for reading Base64-encoded Thrift-serialized records from stdin
  source = "pubsub"

  # Sinks currently supported are:
  # 'kinesis' for writing enriched events to one Kinesis stream and invalid events to another.
  # 'kafka' for writing enriched events to one Kafka topic and invalid events to another.
  # 'pubsub' for writing enriched events to one Cloud Pubsub topic and invalid events to another.
  # 'stdouterr' for writing enriched events to stdout and invalid events to stderr.
  #    Using "sbt assembly" and "java -jar" is recommended to disable sbt
  #    logging.
  sink = "pubsub"

  # AWS credentials
  #
  # If both are set to 'default', use the default AWS credentials provider chain.
  #
  # If both are set to 'iam', use AWS IAM Roles to provision credentials.
  #
  # If both are set to 'env', use environment variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
  aws {
    access-key: "iam"
    secret-key: "iam"
  }

  # Kafka configuration
  kafka {
    brokers: "{{enrichKafkaBrokers}}"
  }

  pubsub {
    projectId: "engineering-sandbox"
  }

  streams {
    in: {
      raw: "raw-sub"

      # Maximum number of records to get from Kinesis per call to GetRecords
      maxRecords: 10000

      # After enrichment, are accumulated in a buffer before being sent to Kinesis.
      # The buffer is emptied whenever:
      # - the number of stored records reaches record-limit or
      # - the combined size of the stored records reaches byte-limit or
      # - the time in milliseconds since it was last emptied exceeds time-limit when
      #   a new event enters the buffer
      buffer: {
        byte-limit: 4000000
        record-limit: 500 # Not supported by Kafka; will be ignored
        time-limit: 5000
      }
    }

    out: {
      enriched: "good"
      bad: "bad"

      # Minimum and maximum backoff periods
      # - Units: Milliseconds
      backoffPolicy: {
        minBackoff: 6000
        maxBackoff: 300
      }
    }

    # "app-name" is used for a DynamoDB table to maintain stream state.
    # "app-name" is used as the Kafka consumer group ID.
    # You can set it automatically using: "SnowplowKinesisEnrich-$\\{enrich.streams.in.raw\\}"
    app-name: "test-app-name"

    # LATEST: most recent data.
    # TRIM_HORIZON: oldest available data.
    # Note: This only effects the first run of this application
    # on a stream.
    initial-position = "TRIM_HORIZON"

    region: "{{enrichStreamsRegion}}"
  }

  # Optional section for tracking endpoints
  monitoring {
    snowplow {
      collector-uri: "0.0.0.0"
      collector-port: 8080
      app-id: "pubsub-test"
      method: "GET"
    }
  }
}

```

You will also need an iglu resolver, and enrichments. These files can be stored both in the same machine where your collector will run or in Cloud Datastore, under any entity with a "json" property. To add, for example, the iglu resolver, go to https://console.cloud.google.com/datastore/entities/query?project=YOUR-PROJECT-ID , click "Create Entity", fill in its Kind (we used "resolver") and introduce its name/id manually, or take note of the auto-generated one after you click "Create".

[[images/gcloud/iglu-resolver.png]]

[[images/gcloud/iglu-resolver2.png]]

Then, when running the project, pass the resolver parameter as: ```--resolver googleds:yourProjectId/resolver/resolver_name_or_id```

<a name="running" >
### 4. Running Stream Enrich
- Download the Stream Enrich fat-jar from Bintray. (The download destination will depend on where you want
to run the Collector)
- To run Stream Enrich , you'll need a config file as the one above.
- You'll also want to authenticate the machine where the collector will run by doing:
``` 
$ gcloud auth login 
$ gcloud application-default auth login
```

NOTE: If you're running Stream Enrich on a Compute Instance, you don't need to authenticate with the above commands, you just need to set the appropriate permissions for your service accounts (automatically authenticated in Compute Instances), so that they're allowed to use Pub/Sub.

<a name="running-locally" >
#### 4a. locally (useful for testing)
To run the collector locally, assuming you have the above files in place,
simply run:
```
$ ./stream-enrich --config config.hocon --resolver file:path/to/iglu_resolver.json --enrichments file:path/to/enrichments
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
    
[[images/gcloud/gcloud-instance-create1.png]]

[[images/gcloud/gcloud-instance-create2.png]]

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

```

Ssh into your instance:

```
$ gcloud compute ssh your-instance-name --zone your-instance-zone
```

And then run the following commands, to install needed programs to run Stream Enrich:
```
$ sudo apt-get update
$ sudo apt-get -y install default-jre
$ sudo apt-get -y install unzip
```

Then use ```wget``` to download the Stream Enrich fat jar to your instance, use ```unzip``` to unzip it and ```chmod +x``` to make the jar executable. Create a config file like the example above and run Stream Enrich as you would locally