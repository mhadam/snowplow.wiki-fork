**This is pre-release**

## Contents

- 1. [Introduction](#intro)
- 2. [Enable and Setup Google Cloud Pub/Sub](#pubsub)
- 3. [Setting up the Scala Stream Collector locally](#ssc)
  

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
[[/images/gcloud-enable-pubsub.png]]

- You'll then have to create the topics to which the Scala Stream Collector publishes:
	* Click on the hamburger, on the top left corner
	* Type "Pub/Sub" or scroll down until you find it, under "Big Data"
	[[/images/gcloud-pubsub-sidebar.png]]
	* Create two topics: these will be the good topic and bad topic.
	[[/images/gcloud-pubsub-topics.png]]


<a name="ssc">
### 3. Setting up the Scala Stream Collector

- To set up the Scala Stream Collector, fill the apropriate fields of the config file (check the [example](https://raw.githubusercontent.com/snowplow/snowplow/pubsub-collector/2-collectors/scala-stream-collector/examples/config.hocon.sample))
	* sink.enabled = "pubsub"
	* sink.pubsub.topic.good = "GOOD-SINK-NAME" _in the case of this tutorial, this will be 'good'_
	* sink.pubsub.topic.bad = "BAD-SINK-NAME" _in the case of this tutorial, this will be 'bad'_
	* sink.pubsub.google-project-id = "YOUR-PROJECT-ID" _in the case of this tutorial, this will be 'example-project-156611'_
	* sink.pubsub.google-auth-path = "/absolute/path/to/credentials/file.json" _refer to [this](https://github.com/snowplow/snowplow/wiki/Snowplow-on-Google-Cloud:-Getting-Started#4-authentication-and-credentials) for more info on what this file is and how to get it_

- Just run it! :)
