[**HOME**](Home) > **SNOWPLOW GLOSSARY OF TERMS**

<a name="top" />

----------

[A](#A) - [B]() - [C](#C) - [D](#D) - [E](#E) - [F]() - [G]() - [H](#H) - [I](#I) - [J]() - [K]() - [L]() - [M]() - [N]() - [O]() - [P](#P) - [Q]() - [R]() - [S](#S) - [T](#T) - [U](#U) - [V]() - [W](#W) - [X]() - [Y]() - [Z]()

----------

<a name="A" />
<a name="g-analytics" />

#### Analytics

Once we have our data [modeled](#g-data-modeling) in tidy users, sessions, content items tables, we are ready to perform analysis on them.

Most companies that use Snowplow will perform analytics using a number of different types of tools:

1. It is common to implement a Business Intelligence tool on top of Snowplow data to enable users (particularly non-technical users) to slice and dice (pivot) on the data. For many companies, the BI tool will be the primary way that most users interface with Snowplow data.
2. Often a data scientist or data science team will often crunch the underlying event-level data to perform more sophisticated analysis including building predictive models, perform marketing attribution etc. The data scientist(s) will use one or more specialist tools e.g. Python for Data Science or R.

*Read [more][analytics] or go to the [top](#top)*

<a name="C" />
<a name="g-collector" />

#### Collector

A collector receives data in the form of `GET` or `POST` requests from the [trackers](#g-tracker), and write the data to either logs or streams (for further processing).

*Read [more](collectors) or go to the [top](#top)*

<a name="g-context" />

#### Context

A context is the group of [entities](#g-entity) associated with or describing the setting in which an [event](#g-event) has taken place. What makes contexts interesting is that they are common across multiple different event types. Thus, contexts provide a convenient way in Snowplow to schema common [entities](#g-entity) once, and then use those schemas across all the different [events](#g-event) where those [entities](#g-entity) are relevant.

Across all our [trackers](#g-tracker), the approach is the same. Each context is a [self-describing JSON](#g-self-describing-json). We create an array of all the different contexts that we wish to pass into Snowplow, and then we pass those contexts in generally as the final argument on any track method that we call to capture the [event](#g-event).

*Read [more][contexts] or go to the [top](#top)*

<a name="D" />
<a name="g-data-analysis" />

#### Data analysis

*See [**Analytics**](#g-analytics)*

<a name="g-data-collection" />

#### Data collection

At data collection time, we aim to capture all the data required to accurately represent a particular [event](#g-event) that has just occurred.

At this stage, the data that is collected should describe the events as they have happened, including as much rich information about:

1. The [event](#g-event) itself
2. The individual/entity that performed the action - that individual or [entity](#g-entity) is a "context"
3. Any "objects" of the action - those objects are also "context"
4. The wider [context](#g-context) that the [event](#g-event) has occurred in

For each of the above we want to collect as much data describing the [event](#g-event) and associated [contexts](#g-context) as possible.

*Read [more][data-collection] or go to the [top](#top)*

<a name="g-data-enrichment" />

#### Data enrichment

*See [**Enrichment**](#g-enrichment)*

<a name="g-data-modeling" />

#### Data modeling

The data [collection](#g-data-collection) and [enrichment](#g-enrichment) process generates a data set that is an "event stream": a long list of packets of data, where each packet represents a single event.

Whilst it is possible to do analysis directly on this event stream, it is very common to:

1. Join the event-stream data set with other data sets (e.g. customer data, product data, media data, marketing data, financial data).
2. Aggregate the event-level data into smaller data sets that are easier and faster to run analyses against.
3. Apply "business logic" i.e. definitions to the data as part of that aggregation step.

What tables are produced, and the different fields available in each, varies widely between companies in different sectors, and surprisingly even varies within the same vertical. That is because part of putting together these aggregate tables involves implementing business-specific logic.

We call this process of aggregating a "data modeling". At the end of the data modeling process, a clean set of tables is available to make it easier to perform analysis on the data.

*Read [more][data-modeling] or go to the [top](#top)*

<a name="E" />
<a name="g-entity" />

#### Entity

In [event](#g-event) modeling terms, an entity is a thing or object which is somehow relevant to the event that we are observing. We use the word "entity" because the word "object" is too loaded - it has too many connotations.

There is a lot of confusion around the role of entities within [events](#g-event) - even to the extent of one analytics company arguing that entity data is completely distinct from event data. In fact nothing could be further from the truth - as we see it, our [events](#g-event) consist of *almost nothing but entities*.

*Read [more][entities] or go to the [top](#top)*

<a name="g-event" />

#### Event

An event is something that occurred in a particular point in time. Examples of events include:

- Load a web page
- Add an item to basket
- Enter a destination
- Check a balance
- Search for an item
- Share a video

Snowplow is an event analytics platform. Once you have setup one or more Snowplow trackers, every time an event occurs, Snowplow should spot the event, generate a packet of data to describe the event, and send that event into the Snowplow data [pipeline](#g-pipeline).

*Read [more][events] or go to the [top](#top)*

#### Event Dictionary

When we set up Snowplow, we need to make sure that we track all the [events](#g-event) that are meaningful to our business, so that the data associated with those events is available in Snowplow for analysis.

When we come to analyse Snowplow data, we need to be able to look at the event data and understand, in an unambiguous way, what that data actually means i.e. what it represents.

An event dictionary is a crucial tool in both cases. It is a document that defines the universe of events that a company is interested in tracking.

*Read [more][event-dictionary] or go to the [top](#top)*

<a name="g-enrichment" />

#### Enrichment

Data enrichment is sometimes referred to as "dimension widening". We are using 3rd party sources of data to enrich the data we originally collected about the event so that we have more context available for understanding that event, enabling us to perform richer [analysis](#g-analytics).

[Snowplow](#g-snowplow) supports the following enrichments out-of-the-box. We are working on making our enrichment framework pluggable, so that users and partners can extend the list of enrichments performed as part of the data processing pipeline:

1. IP -> Geographic location
2. Referrer query string -> source of traffic
3. User agent string -> classifying devices, operating systems and browsers

*Read [more][enrichment] or go to the [top](#top)*

<a name="H" />
<a name="g-huskimo " />

#### Huskimo

Huskimo is an open-source product from the Snowplow team. It connects to third-party SaaS platforms (e.g [Singular](https://www.singular.net/), [Twilio](https://www.twilio.com/)), exports their data via API, and then uploads that data into your Redshift instance. Huskimo has a simple goal: to make essential datasets currently locked away inside various SaaS platforms available for [analysis](#g-analytics) inside Redshift.

Huskimo is a complement to Snowplow's built-in [webhook](#g-webhook) support. It came about because not all SaaS services offer [webhooks](#g-webhook) which expose their internal data as a stream of [events](#g-event). Note that you do not need to use Snowplow to use Huskimo.

*Read [more][huskimo] or go to the [top](#top)*

<a name="I" />
<a name="g-iglu" />

#### Iglu

Iglu is a machine-readable, open-source *schema repository* for [JSON Schema](http://json-schema.org/) from the team at Snowplow Analytics. A schema repository (sometimes called a registry) is like *npm* or *Maven* or *git* but holds data schemas instead of software or code.

Snowplow uses Iglu to store all the schemas associated with the different [events](#g-event) and [contexts](#g-context) that are captured via Snowplow. When an [event](#g-event) or [context](#g-context) is sent into Snowplow, it is sent with a reference to the schema for the [event](#g-event) or [context](#g-context), which points to the location of the schema for the [event](#g-event) or [context](#g-context) in Iglu.

*Read [more][iglu] or go to the [top](#top)*

<a name="P" />
<a name="g-pipeline" />

#### Pipeline

The Snowplow pipeline is built to enable a very clean separation of the following steps in the *data processing flow*:

1. [Data collection](#g-data-collection)
2. [Data enrichment](#g-enrichment)
3. [Data modelling](#g-data-modeling)
4. [Data analysis](#g-analytics)

*Read [more][pipeline] or go to the [top](#top)*

<a name="S" />
<a name="g-sauna" />

#### Sauna

Sauna is an open-source decisioning and response framework from Snowplow Analytics team. Analysts and data scientists (and some data engineers) are the end users of Sauna: you want to use Sauna to automate responses to your event streams in third-party systems.

*Read [more][sauna] or go to the [top](#top)*

<a name="g-schema-ddl" />

#### Schema DDL

Schema DDL is a set of generators for producing various DDL formats from [JSON Schemas](http://json-schema.org/). It's tightly coupled with other tools from [Snowplow Platform](#g-snowplow) like [Iglu](#g-iglu) and [Self-describing JSON](#g-self-describing-json) and used mostly in [Schema Guru](#g-schema-guru).

*Read [more][schema-ddl] or go to the [top](#top)*

<a name="g-schema-guru" />

#### Schema Guru

Schema Guru is a tool (CLI, Spark job and web) allowing you to derive [JSON Schemas](http://json-schema.org/) from a set of JSON instances process and transform it into different data definition formats.

Current primary features include:

- derivation of JSON Schema from set of JSON instances
- generation of [Redshift](http://aws.amazon.com/redshift/) table DDL and [JSONPaths file](http://docs.aws.amazon.com/redshift/latest/dg/copy-usage_notes-copy-from-json.html)

Unlike other tools for deriving JSON Schemas, Schema Guru allows you to derive schema from an unlimited set of instances (making schemas much more precise), and supports many more JSON Schema validation properties.

Schema Guru is used heavily in association with Snowplow's own [Snowplow](#g-snowplow), [Iglu](#g-iglu) and [Schema DDL](#g-schema-ddl) projects.

*Read [more][schema-guru] or go to the [top](#top)*

<a name="g-schemaver" />

#### SchemaVer

SchemaVer is Snowplow Team own schema versioning notion. It is defined as follows: `MODEL-REVISION-ADDITION`

- `MODEL` when you make a breaking schema change which will prevent interaction with any historical data
- `REVISION` when you introduce a schema change which may prevent interaction with some historical data
- `ADDITION` when you make a schema change that is compatible with all historical data

*Read [more][schemaver] or go to the [top](#top)*

<a name="g-self-describing-json" />

#### Self-describing JSON

Self-describing JSON is an individual [JSON](http://www.json.org/) with its [JSON Schema](http://json-schema.org/). It generally looks like the one below:

```json
{
	"schema": "iglu:com.snowplowanalytics/ad_click/jsonschema/1-0-0",
	"data": {
		"bannerId": "4732ce23d345"
	}
}
```

It differs from [standard JSON](http://www.json.org/) due to the following important changes :

1. We have added a new top-level field, schema, which contains (in a space-efficient format) all the information required to uniquely identify the associated JSON Schema
2. We have moved the JSON’s original property inside a data field. This sandboxing will prevent any accidental collisions should the JSON already have a schema field

*Read [more][self-describing-json] or go to the [top](#top)*

<a name="g-shredding" />

#### Shredding

[Snowplow](#g-snowplow) has a Shredding process (as part of [Enrichment](#g-enrichment) and [Storage](#g-storage) processes) which consists of two phases:

1. Extracting [unstructured event](#g-unstructured-event) JSONs and [context](#g-context) JSONs from [enriched](#g-enrichment) event files into their own files
2. Loading those files into corresponding tables in Redshift

There are three great use cases to use the shredding functionality for:

1. Adding support into your Snowplow installation for *new Snowplow event types* with no software upgrade required - simply add new tables to your Redshift database.
2. Defining your own *custom unstructured events types*, and processing these through the Snowplow [pipeline](#g-pipeline) into dedicated tables in Redshift. Retailers can define their own "product view" or "add to basket" events, for example. Media companies can define their own "video play" events.
3. Defining your own *custom context types*, and processing these through the Snowplow [pipeline](#g-pipeline) into dedicated tables in Redshift. You can define your own "user" type, for example, including whatever fields you capture and want to store related to a user.

*Read [more][shredding] or go to the [top](#top)*

<a name="g-snowplow" />

#### Snowplow

Snowplow is an enterprise-strength marketing and product [analytics](#g-analytics) platform. It does three things:

- Identifies your users, and tracks the way they engage with your website or application
- Stores your users' behavioural data in a scalable "event data warehouse" you control: in Amazon S3 and (optionally) Amazon Redshift or Postgres
- Lets you leverage the biggest range of tools to [analyze](#g-analytics) that data, including big data tools (e.g. Hive, Pig, Mahout) via EMR or more traditional tools e.g. Tableau, R, Looker, Chartio to [analyze](#g-analytics) that behavioural data

*Read [more](http://snowplowanalytics.com/) or go to the [top](#top)*

<a name="g-storage">

#### Storage

The [enrichment](#g-enrichment) process takes raw Snowplow [collector](#g-collector) logs, tidies them up, enriches them (e.g. by adding Geo-IP data, and performing referrer parsing) and then writes the output of that process back to S3 as a cleaned up set of Snowplow event files. The data in these files can be analysed directly by any big data tool that runs on EMR.

In addition, Snowplow data from those event files could be copied into Amazon Redshift, where it can be analysed using any tool that talks to PostgreSQL.

There are therefore a number of different potential storage modules that Snowplow users can store their data in.

*Read [more](Storage-documentation) or go to the [top](#top)*

<a name="g-structured-event" />

#### Structured event

We follow Google five-variable tracking event structure. When you track a structured event, you get five parameters:

- *Category*: The name for the group of objects you want to track.
- *Action*: A string that is used to define the user in action for the category of object.
- *Label*: An optional string which identifies the specific object being actioned.
- *Property*: An optional string describing the object or the action performed on it.
- *Value*: An optional numeric data to quantify or further describe the user action.

For example, when tracking a custom structured event the specification for the `trackStructEvent` method (Javascript tracker) would follow the pattern:

```
snowplow_name_here('trackStructEvent', 'category','action','label','property','value');
```

*Read [more](Canonical-event-model#customstruct) or go to the [top](#top)*

<a name="T" />
<a name="g-tracker" />

#### Tracker

A tracker is client- or server-side libraries which track customer behaviour by sending Snowplow [events](#g-event) to a [Collector](#g-collector).

*Read [more](trackers) or go to the [top](#top)*

<a name="U" />
<a name="g-unstructured-event" />

#### Unstructured event

You may wish to track [events](#g-event) on your website or application which are not directly supported by Snowplow and which [structured event](#g-structured-event) tracking does not adequately capture. Your [event](#g-event) may have more than the five fields offered by `trackStructEvent`, or its fields may not fit into the *category-action-label-property-value* model. The solution is Snowplow's custom unstructured events. Unstructured events use [self-describing JSON](#g-self-describing-json) which can have arbitrarily many fields.

For example, to track an unstructured event with Javascript tracker, you make use of the `trackUnstructEvent` method with the pattern shown below:

```
snowplow_name_here('trackUnstructEvent', <<SELF-DESCRIBING EVENT JSON>>);
```

*Read [more][unstructured-events] or go to the [top](#top)*

<a name="W" />
<a name="g-webhook" />

#### Webhook

Snowplow allows you to collect [events](#g-event) via the adapters (webhooks) of supported third-party software.

Webhooks allow this third-party software to send their own internal event streams to Snowplow [collectors](#g-collector) for further processing. Webhooks are sometimes referred to as "streaming APIs" or "HTTP response APIs".

*Read [more](Setting-up-a-webhook) or go to the [top](#top)*


[analytics]: http://snowplowanalytics.com/documentation/concepts/snowplow-data-pipeline/#data-analysis
[contexts]: http://snowplowanalytics.com/documentation/concepts/contexts
[data-collection]: http://snowplowanalytics.com/documentation/concepts/snowplow-data-pipeline/#data-collection
[data-modeling]: http://snowplowanalytics.com/documentation/concepts/snowplow-data-pipeline/#data-modeling
[entities]: http://snowplowanalytics.com/blog/2015/01/18/modeling-events-through-entity-snapshotting/
[enrichment]: http://snowplowanalytics.com/documentation/concepts/snowplow-data-pipeline/#data-enrichment
[events]: http://snowplowanalytics.com/documentation/concepts/events
[event-dictionary]: http://snowplowanalytics.com/documentation/concepts/event-dictionaries-and-schemas
[huskimo]: http://snowplowanalytics.com/blog/2015/08/30/huskimo-0.3.0-released-warehouse-your-twilio-telephony-data-in-redshift/
[iglu]: http://snowplowanalytics.com/documentation/concepts/iglu
[pipeline]: http://snowplowanalytics.com/documentation/concepts/snowplow-data-pipeline
[sauna]: https://github.com/snowplow/sauna/wiki
[schema-ddl]: https://github.com/snowplow/schema-ddl
[schema-guru]: https://github.com/snowplow/schema-guru
[schemaver]: https://github.com/snowplow/iglu/wiki/SchemaVer
[self-describing-json]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
[shredding]: https://github.com/snowplow/snowplow/wiki/Shredding
[unstructured-events]: http://snowplowanalytics.com/blog/2013/05/14/snowplow-unstructured-events-guide/
