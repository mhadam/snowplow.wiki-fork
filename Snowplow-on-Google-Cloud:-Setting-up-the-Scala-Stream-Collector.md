**This is pre-release**

## Contents

- 1. [Introduction](#intro)
- 2. [Enable and Setup Google Cloud Pub/Sub](#pubsub)
- 3. [Setting up the Scala Stream Collector locally](#ssc-locally)
- 4. [Setting up the Scala Stream Collector on a Google Compute instance](#ssc-instance)
  

<a name="intro">
### 1. Introduction

The Scala Stream Collector can have Google Cloud Pub/Sub as its sink. Pub/Sub is a distributed Message Queue,
implemented completely on Google's structure. Publisher applications publish to topics, whilst subscriber
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


<a name="ssc-locally">
### 3. Setting up the Scala Stream Collector locally

<a name="ssc-instance">
### 4. Setting up the Scala Stream Collector on a Google Compute instance
