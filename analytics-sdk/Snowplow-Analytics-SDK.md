[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » Snowplow Analytics SDK

## Overview

We are pleased to announce the releases of our first analytics SDKs for Snowplow, created for data engineers and data scientists working with Snowplow in Scala.

Some good use cases for the SDK include:

1. Performing [event data modeling](http://snowplowanalytics.com/blog/2016/03/16/introduction-to-event-data-modeling/) in Apache Spark as part our Spark batch pipeline
2. Developing machine learning models on your event data using Apache Spark (e.g. using [Databricks](https://databricks.com/) or [Zeppelin on EMR](https://blogs.aws.amazon.com/bigdata/post/Tx6J5RM20WPG5V/Building-a-Recommendation-Engine-with-Spark-ML-on-Amazon-EMR-using-Zeppelin))
3. Performing analytics-on-write in AWS Lambda as part of our Kinesis real-time pipeline:

![Snowplow Analytics SDK use cases](http://snowplowanalytics.com/assets/img/blog/2016/03/scala-analytics-sdk-usage.png)

We are hugely excited about developing our analytics SDK initiative in four directions:

1. Adding more SDKs for other languages popular for data analytics and engineering, including Python, Node.js (for AWS Lambda) and Java
2. Adding additional event transformers to the Scala Analytics SDK - please let us know any suggestions!
3. We are planning on “dogfooding” the Scala Analytics SDK by starting to use it in standard Snowplow components, such as our Kinesis Elasticsearch Sink
4. Adding additional functions that are useful for processing event data (and sequences of event data) in particular

## Snowplow Analytics SDKs

- [Scala Analytics SDK](Scala-Analytics-SDK) - lets you work with Snowplow enriched events in your Scala event processing, data modeling and machine-learning jobs. You can use this SDK with Apache Spark, AWS Lambda, Apache Flink, Scalding, Apache Samza and other Scala-compatible data processing frameworks.
- [Python Analytics SDK](Python-Analytics-SDK) - lets you work with Snowplow enriched events in your Python event processing, data modeling and machine-learning jobs. You can use this SDK with Apache Spark, AWS Lambda, and other Python-compatible data processing frameworks.
- Node.js Analytics SDK
- Java Analytics SDK
