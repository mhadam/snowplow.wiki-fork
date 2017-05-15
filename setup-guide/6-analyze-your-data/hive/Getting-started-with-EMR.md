[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 6: Get started analyzing Snowplow data**](Getting-started-analyzing-Snowplow-data) > [**Getting started with EMR and Hive**](Getting-started-with-EMR)

## An overview of Amazon Elastic Mapreduce

Elastic Mapreduce (EMR) is a service provided by Amazon that makes it relatively straight forward to use Spark and Spark-powered tools (e.g. Hive, MLLib, Mahout) to process data stored in S3. Amazon has made it easy to spin up clusters of machines of varying sizes to process large volumes of data, and spin them down once querying is complete. 

Because Snowplow data is stored on S3 and Snowplow data volumes are often very large, processing them using Spark on EMR is a particularly attractive option for a wealth of analysis. This is especially true for people who wish to run machine learning algorithms on the Snowplow data set using Mahout. (E.g. to develop recommendation engines or segment audience by behaviour.)

More details on EMR can be found on the [Amazon website](http://aws.amazon.com/elasticmapreduce/).



