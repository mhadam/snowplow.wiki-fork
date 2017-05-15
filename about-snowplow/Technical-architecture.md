Snowplow has a very different architecture from conventional open-source web analytics packages such as [Piwik][piwik] or [Open Web Analytics][owa]. Where those packages are built on a tightly-coupled LAMP stack, Snowplow has a loosely-coupled architecture which consists of six sub-systems:

![architecture][conceptual-architecture]

To briefly explain these sub-systems:

* **Trackers** fire Snowplow events. Currently we have 12 trackers, covering web, mobile, desktop, server and IoT. (For more information see the [trackers section](https://github.com/snowplow/snowplow/tree/master/1-trackers) of the repository). Additionally, **webhooks** allow third-party software to send their own internal event streams to Snowplow Collectors for further processing. [Webhooks](Setting-up-a-webhook) are sometimes referred to as "streaming APIs" or "HTTP response APIs".
* **Collectors** receive Snowplow events from trackers. Currently we have three different event collectors, sinking events either to Amazon S3 or Amazon Kinesis, namely a CDN-based [Cloudfront Collector](https://github.com/snowplow/snowplow/tree/master/2-collectors/cloudfront-collector) on [Amazon CloudFront][cloudfront], a collector that sets a third party pixel for cross-domain tracking called the [Clojure Collector](https://github.com/snowplow/snowplow/tree/master/2-collectors/clojure-collector), and a [Scala Stream Collector](https://github.com/snowplow/snowplow/tree/master/2-collectors/scala-stream-collector) which sets a third-party cookie for cross-domain tracking.
* **Enrichment** cleans up the raw Snowplow events, enriches them and puts them into storage. Currently we have a Spark-based enrichment process, and a Kinesis-based process.
* **Storage** is where the Snowplow events live. Currently we store the Snowplow events in an S3, Amazon Redshift and PostgreSQL.
* **Data modeling** is where event-level data is joined with other data sets and aggregated into smaller data sets, and business logic is applied. This produces a clean set of tables which make it easier to perform analysis on the data. We have data models for Redshift and [Looker](http://www.looker.com/).
* **Analytics** are performed on the Snowplow events or on the aggregate tables. We currently have an online cookbook of ad hoc analyses that work with Redshift, Postgres and Hive. We also have data models for [Looker](http://www.looker.com/) in LookML.

In the rest of this page we explain our rationale for this architecture, map out the specific technical components and finally flag up the strengths and limitations of this architecture.

## Rationale for architecture

Snowplow's distinctive architecture has been informed by a set of key design principles:

1. **Extreme scalability** - Snowplow should be able to scale to tracking billions of customer events without affecting the performance of your client (e.g. website) or making it difficult to subsequently analyse all of those events
2. **Permanent event history** - Snowplow events should be stored in a simple, non-relational, immutable data store
3. **Direct access to individual events** - you should have direct access to your raw Snowplow event data at the atomic level
4. **Separation of concerns** - event tracking and event analysis should be two separate systems, only loosely-coupled
5. **Support any analysis** - Snowplow should make it easy for business analysts, data scientists and engineers to answer any business question they want, using as wide a range of analytical tools as possible

## Technical strengths

The Snowplow approach has several technical advantages over more
conventional web analytics approaches. In no particular order, these
advantages are:

* **Scalable, fast tracking** - using CloudFront for event tracking
    reduces complexity and minimizes client slowdown worldwide
* **Never lose your raw data** - your raw event data is never
    compacted, overwritten or otherwise corrupted by Snowplow
* **Direct access to events** - not intermediated by a third-party
    vendor, or a slow API, or an interface offering aggregates only
* **Analysis tool agnostic** - Snowplow can be used to feed whatever
    analytics process you want (e.g. Hive, R, Pig, Sky EQL)
* **Integrable with other data sources** - join Snowplow data into
    your other data sources (e.g. ecommerce, CRM) at the event level
* **Clean separation of tracking and analysis** - new analyses will not
    require re-tagging of your site or app

## Technical limitations

The current Snowplow architecture, tightly coupled as it is to Amazon
CloudFront and S3, has some specific limitations to consider:

* **Not real-time** - CloudFront takes 20-60 minutes to collate logs from its edge nodes, so real-time analytics are not feasible. In addition the enrichment process is batch-based, rather than stream-based
* **Data payload limited by querystring length** - Snowplow data logged via a `GET` querystring could potentially hit the de facto [2000 character][2000char] URL length limit

For more information on these limitations, please see the [[Technical FAQ|Developer-FAQ]].

However, the limitations above have been lifted with the release of [Scala Stream Collector](Scala-Stream-Collector) and [Stream Enrich](Stream-Enrich), both of which are Amazon Kinesis-based. Additionally, both [Scala Stream Collector](Scala-Stream-Collector) and [Clojure Collector](Clojure-collector) support `POST` queries, which can potentially accommodate [unlimited data size][post-limits].

[conceptual-architecture]: https://camo.githubusercontent.com/05914f02874cfc540e98af29bd68bf0d6818f54e/68747470733a2f2f64336936666d7331636d316a30692e636c6f756466726f6e742e6e65742f6769746875622d77696b692f696d616765732f736e6f77706c6f772d6172636869746563747572652e706e67
[conceptual-architecture-old]: https://d3i6fms1cm1j0i.cloudfront.net/github-wiki/images/conceptual-architecture.png
[tech-architecture]: https://d3i6fms1cm1j0i.cloudfront.net/github-wiki/images/technical-architecture.png
[piwik]: http://piwik.org/
[owa]: http://www.openwebanalytics.com/
[cloudfront]: http://aws.amazon.com/cloudfront/
[s3]: http://aws.amazon.com/s3/
[hive]: http://hive.apache.org/
[2000char]: http://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url
[post-limits]: http://stackoverflow.com/questions/2880722/is-http-post-limitless
