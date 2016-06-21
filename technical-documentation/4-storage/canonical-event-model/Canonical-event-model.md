[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow technical documentation) > [**Storage**](storage documentation) > Canonical Event Model

<a name="top" />
# Canonical event model

This relates to the Canonical Event Model from Snowplow Release 73 Cuban Macaw onwards. For previous versions:

* [Canonical Event Model pre-Release 73](Canonical-event-model-v72)
* [Canonical Event Model pre-Release 71](Canonical-event-model-v70)
* [Canonical Event Model pre-Release 63](Canonical-event-model-pre-r63)

Table of contents:

1. [Overview](#overview)  
2. [The Snowplow canonical data structure: understanding the individual fields](#model)  
3. [A note about data storage formats](#note)  
4. [Specific unstructured events and custom contexts](#specific-unstruct-events-contexts)  

<a name="overview" />
## 1. Overview

In order to analyse Snowplow data, it is important to understand how it is structured. We have tried to make the structure of Snowplow data as simple, logical, and easy-to-query as possible.

* **Each line represents one event**. Each line in the Snowplow events table represents a single *event*, be that a *page view*, *add to basket*, *play video*, *like* etc.
* **Structured data**. Snowplow data is structured: individual fields are stored in their own columns, making writing sophisticated queries on the data easy, and making it straightforward for analysts to plugin any kind of analysis tool into their Snowplow data to compose and execute queries
* **Extensible schema**. Snowplow started life as a web analytics data warehousing platform, and has a basic schema suitable for performing web analytics, with a wide range of web-specific dimensions (related to page URLs, browsers, operating systems, devices, IP addresses, cookie IDs) and web-specfic events (page views, page pings, transactions). All of these fields can be found in the `atomic.events` table, which is a "fat" (many columns) table. As Snowplow has evolved into a general purpose event analytics platform, we've enabled Snowplow users to define additional event types (we call these *custom unstructured events*) and define their own entities (we call these *custom contexts*) so that they can extend the schema to suit their own businesses. For Snowplow users running Amazon Redshift, each custom unstructured event and custom context will be stored in its own dedicated table, again with one line per event. These additional tables can be joined back to the core `atomic.events` table, by joining on the `root_id` field in the custom unstructured event / custom context table with the `event_id` in the `atomic.events` table
* **Single table**. All the events are effectively stored in a single table, making running queries across the data very easy. Even if you're running Snowplow with Redshift and have extended the schema as described above, you can still query the data as if it were in a single fat table. This is because:
  * The joins from the additional tables to the core `atomic.events` table are one-to-one
  * The field joined on is the distribution and sort key for both tables, so queries are as fast as if the data were in a single table
* **Immutable log**. The Snowplow data table is designed to be immutable: the data in each line should not change over time. Data points that we would expect to change over time (e.g. what cohort a particular user belongs to, how we classify a particular visitor) can be derived from Snowplow data. However, our recommendation is that these derived fields should be defined and calculated at analysis time, stored in a separate table and joined to the *Snowplow events table* when performing any analysis

<a name="model" />
## 2. The Snowplow canonical data structure: understanding the individual fields

- 2.1 [**Common fields (platform and event independent)**](#common)  
  - 2.1.1 [Application fields](#application)  
  - 2.1.2 [Date / time fields](#date-time)  
  - 2.1.3 [Event / transaction fields](#eventtransaction)  
  - 2.1.4 [Snowplow version fields](#version)  
  - 2.1.5 [User-related fields](#user)  
  - 2.1.6 [Device and operating system fields](#device)  
  - 2.1.7 [Location fields](#location)
  - 2.1.8 [IP address-based fields](#ip)
  - 2.1.9 [Metadata fields](#metadata)
- 2.2 [**Platform-specific fields**](#platform)  
  - 2.2.1 [Web-specific fields](#web)  
- 2.3 [**Event-specific fields**](#event)
  - 2.3.1 [Page views](#pageview)  
  - 2.3.2 [Page pings](#pagepings)  
  - 2.3.3 [Ecommerce transations](#ecomm)   
  - 2.3.4 [Error tracking](#error)  
  - 2.3.5 [Custom structured events](#customstruct)  
  - 2.3.6 [Custom unstructured events](#customunstruct)
  - 2.3.7 [Custom contexts](#customcontexts)
- 2.4 [**Specific unstructured events**](#specific-unstruct-events-contexts)

<a name="common" />
### 2.1 Common fields (platform and event independent)

<a name="application" />
#### 2.1.1 Application fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:---------------|
| `app_id`        | text     | Application ID  | Yes       | 'angry-birds'  |
| `platform`      | text     | Platform        | Yes       | 'web'          |    

The application ID is used to distinguish different applications that are being tracked by the same Snowplow stack, e.g. `production` versus `dev`.

The platform ID is used to distinguish the same app running on different platforms, e.g. `iOS` vs `web`.

Back to [top](#top).

<a name="date-time" />
#### 2.1.2 Date / time fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:---------------|
| `collector_tstamp`| timestamp | Time stamp for the event recorded by the collector | Yes    | '2013-11-26 00:02:05'   |
| `dvce_created_tstamp`   | timestamp | Timestamp event was recorded on the client device | No | '2013-11-26 00:03:57.885' |
| `dvce_sent_tstamp` | timestamp | When the event was sent by the client device | No | '2013-11-26 00:03:58.032' |
| `etl_tstamp`   | timestamp | Timestamp event began ETL | No | '2017-01-26 00:01:25.292' |
| `os_timezone`   | text     | Client operating system timezone | No | 'Europe/London' |
| `derived_tstamp` | timestamp | Timestamp making allowance for innaccurate device clock | No | '2013-11-26 00:02:04' |
| `true_tstamp` | timestamp | User-set "true timestamp" for the event | No | '2013-11-26 00:02:04' |

Back to [top](#top).

<a name="eventtransaction" />
#### 2.1.3 Event / transaction fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:---------------|
| `event`         | text     | The type of event recorded | Yes | 'page_view'    |
| `event_id`      | text     | A UUID for each event | Yes | 'c6ef3124-b53a-4b13-a233-0088f79dcbcb' |
| `txn_id`        | int      | Transaction ID set client-side, used to de-dupe records | No | 421828 |
| `event_fingerprint`        | tesxt      | Hash client-set event fields | No | AADCE520E20C2899F4CED228A79A3083 |

A complete list of event types is given [here] (#event).

Back to [top](#top).

<a name="version" />
#### 2.1.4 Snowplow version fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:---------------|
| `v_tracker`     | text     | Tracker version | Yes       | 'no-js-0.1.0'  |
| `v_collector`   | text     | Collector version | Yes     | 'clj-tomcat-0.1.0', 'cf' |
| `v_etl`         | text     | ETL version     | Yes       | 'serde-0.5.2'  |
| `name_tracker`  | text     | Tracker namespace     | No        | 'cloudfront-1' |
| `etl_tags`      | text     | JSON of tags for this ETL run | No | "['prod']" |

Some Snowplow Trackers allow the user to name each specific Tracker instance. `name_tracker` corresponds to this name, and can be used to distinguish which tracker generated which events.

Back to [top](#top).

<a name="user" />
#### 2.1.5 User-related fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:---------------|
| `user_id`       | text     | Unique ID set by business | No | 'jon.doe@email.com' |
| `domain_userid` | text     | User ID set by Snowplow using 1st party cookie | No | 'bc2e92ec6c204a14' |
| `network_userid`| text     | User ID set by Snowplow using 3rd party cookie | No | 'ecdff4d0-9175-40ac-a8bb-325c49733607' |
| `user_ipaddress` | text    | User IP address | No       | '92.231.54.234' |
| `domain_sessionidx` | int      | A visit / session index | No | 3              |
| `domain_sessionid`  | text      | A visit / session identifier | No | 'c6ef3124-b53a-4b13-a233-0088f79dcbcb'              |

`domain_sessionidx` is the number of the current user session. For example, an event occurring during a user's first session would have `domain_sessionidx` set to 1. The JavaScript Tracker calculates this field by storing a visit count in a first-party cookie. Whenever the Tracker fires an event, if more than 30 minutes have elapsed since the last event, the visitor count is increased by 1. (Whenever an event is fired, a "session cookie" is created and set to expire in 30 minutes. This is how the Tracker can tell whether the visit count should be incremented.) Thirty minutes is the default value and can be changed using the `setSessionCookieTimeout` method.


Back to [top](#top).

<a name="device" />
#### 2.1.6 Device and operating system fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:---------------|
| `useragent`     | text     | Raw useragent   | Yes       |                |
| `dvce_type`     | text     | Type of device  | No        | 'Computer'     |
| `dvce_ismobile` | boolean  | Is the device mobile? | No  | 1              |
| `dvce_screenheight` | int  | Screen height in pixels | No | 1024           |
| `dvce_screenwidth`  | int  | Screen width in pixels  | No | 1900           |
| `os_name`       | text     | Name of operating system | No | 'Android'      | 
| `os_family`     | text     | Operating system family | No | 'Linux'        |
| `os_manufacturer` | text   | Company responsible for OS | No | 'Apple'        |

Back to [top](#top).

<a name="location" />
#### 2.1.7 Location fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:---------------|
| `geo_country`   | text     | ISO 3166-1 code for the country the visitor is located in | No  | 'GB', 'US' |
| `geo_region`    | text     | ISO-3166-2 code for country region the visitor is in | No | 'I9', 'TX' |
| `geo_city`      | text     | City the visitor is in | No | 'New York', 'London'       |
| `geo_zipcode`  | text     | Postcode the visitor is in | No | '94109'           |
| `geo_latitude`  | text     | Visitor location latitude | No | 37.443604      |
| `geo_longitude` | text     | Visitor location longitude | No | -122.4124      |
| `geo_region_name` | text     | Visitor region name | No | 'Florida'      |
| `geo_timezone` | text     | Visitor timezone name | No | 'Europe/London'      |

<a name="ip" />
#### 2.1.8 IP address-based fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:---------------|
| `ip_isp`   | text     | Visitor's ISP | No  | 'FDN Communications' |
| `ip_organization`    | text     | Organization associated with the visitor's IP address - defaults to ISP name if none is found | No | 'Bouygues Telecom' |
| `ip_domain`      | text     | Second level domain name associated with the visitor's IP address | No | 'nuvox.net'       |
| `ip_netspeed`  | text     | Visitor's connection type | No | 'Cable/DSL'           |

<a name="metadata" />
#### 2.1.9 Metadata fields

Fields containing information about the event type.

| **Field**       | **Type** | **Description**         | **Reqd?** | **Example**    |
|:----------------|:---------|:------------------------|:----------|:---------------|
| `event_vendor`  | text     | Who defined the event   | No        | 'com.acme' |
| `event_name`    | text     | Event name              | No        | 'link_click' |
| `event_format`  | text     | Format for event        | No        | 'jsonschema'       |
| `event_version` | text     | Version of event schema | No        | '1-0-2'           |

<a name="platform" />
### 2.2 Platform-specific fields

<a name="web" />
#### 2.2.1 Web-specific fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:---------------|
| **Page fields** |          |                 |           |                |
| `page_url`      | text     | The page URL    | Yes       | 'http://www.example.com'|
| `page_urlscheme`| text     | Scheme aka protocol | Yes   | 'https'        |
| `page_urlhost`  | text     | Host aka domain | Yes       | '“www.snowplowanalytics.com' |
| `page_urlport`  | int      | Port if specified, 80 if not| 80             |
| `page_urlpath`  | text     | Path to page    | No        | '/product/index.html' |
| `page_urlquery` | text     | Querystring     | No        | 'id=GTM-DLRG'  |
| `page_urlfragment` | text  | Fragment aka anchor | No    | '4-conclusion' |
| `page_referrer` | text     | URL of the referrer | No    | 'http://www.referrer.com' |
| `page_title`    | text     | Web page title  | No        | 'Using ChartIO to visualize and interrogate Snowplow data - Snowplow Analytics' |
| `refr_urlscheme`| text     | Referer scheme  | No        | 'http'         |
| `refr_urlhost`  | text     | Referer host    | No        | 'www.bing.com' |
| `refr_urlport`  | int      | Referer port    | No        | 80 |
| `refr_urlpath`  | text     | Referer page path | No      | '/images/search' |
| `refr_urlquery` | text     | Referer URL querystring | No | 'q=psychic+oracle+cards' |
| `refr_urlfragment` | text   | Referer URL fragment | No   |                |
| `refr_medium`   | text     | Type of referer | No | 'search', 'internal' |
| `refr_source`   | text     | Name of referer if recognised | No | 'Bing images' |
| `refr_term`     | text     | Keywords if source is a search engine | No | 'psychic oracle cards' |
| `refr_domain_userid` | text      | The Snowplow domain_userid of the referring website | No | 'bc2e92ec6c204a14' |
| `refr_dvce_tstamp`   | timestamp | The time of attaching the domain_userid to the inbound link | No | '2013-11-26 00:02:05' |
| **Document fields** |      |                 |           |           |                |
| `doc_charset`   | text     | The page’s character encoding | No | , 'UTF-8' |
| `doc_width`     | int      | The page's width in pixels  | No | 1024       |
| `doc_height`    | int      | The page's height in pixels | No | 3000       | 
| **Marketing / traffic source fields** |          |                 |           |           |                |
| `mkt_medium`    | text     | Type of traffic source | No | 'cpc', 'affiliate', 'organic', 'social' |
| `mkt_source`    | text     | The company / website where the traffic came from | No | 'Google', 'Facebook' |
| `mkt_term`      | text     | Any keywords associated with the referrer | No | 'new age tarot decks' |
| `mkt_content`   | text     | The content of the ad. (Or an ID so that it can be looked up.) | No | 13894723 |
| `mkt_campaign`  | text     | The campaign ID | No        | 'diageo-123' |
| `mkt_clickid`  | text     | The click ID | No        | 'ac3d8e459' |
| `mkt_network`  | text     | The ad network to which the click ID belongs | No        | 'DoubleClick' |
| **Browser fields** |          |                 |           |           |                |
| `user_fingerprint` | int   | A user fingerprint generated by looking at the individual browser features | No | 2161814971 |
| `connection_type` | text   | Type of internet connection | No | No   | - |
| `cookie`        | boolean  | Does the browser support persistent cookies? | No | 1 |
| `br_name`       | text     | Browser name  | No | 'Firefox 12' |
| `br_version`    | text     | Browser version  | No | '12.0' |
| `br_family`     | text     | Browser family  | No | 'Firefox' |
| `br_type`       | text     | Browser type  | No| 'Browser' |
| `br_renderengine` | text   | Browser rendering engine  | No | 'GECKO' |
| `br_lang`       | text     | Language the browser is set to  | No | 'en-GB' |
| `br_features_pdf` | boolean     | Whether the browser recognizes PDFs | No | 1 |
| `br_features_flash` | boolean     | Whether Flash is installed | No | 1 |
| `br_features_java` | boolean     | Whether Java is installed | No | 1 |
| `br_features_director` | boolean     | Whether Adobe Shockwave is installed | No | 1 |
| `br_features_quicktime` | boolean     | Whether QuickTime is installed | No | 1 |
| `br_features_realplayer` | boolean     | Whether RealPlayer is installed | No | 1 |
| `br_features_windowsmedia` | boolean     | Whether mplayer2 is installed | No | 1 |
| `br_features_gears` | boolean     | Whether Google Gears is installed | No | 1 |
| `br_features_silverlight` | boolean     | Whether Microsoft Silverlight is installed | No | 1 |
| `br_cookies` | boolean     | Whether cookies are enabled | No | 1 |
| `br_colordepth` | int      | Bit depth of the browser color palette  | No | 24 |
| `br_viewheight`| int     | Viewport height    | No     | 1000 |
| `br_viewwidth` | int     | Viewport width     | No     | 1000 |

See [issue 94](https://github.com/snowplow/snowplow/issues/94) for more details on `br_windowheight` and `br_windowwidth`.

Back to [top](#top).

<a name="event" />
### 2.3 Event-specific fields

Snowplow includes specific fields to capture data associated with specific events.

Note that to date, all event types have been defined by Snowplow. Also note that `event_vendor` values follow the [Java package naming convention](http://docs.oracle.com/javase/tutorial/java/package/namingpkgs.html).

Snowplow currently supports (or will support in the near future) the following event types:

|        | **Event type**                                              | **Value of `event` field in model**    |
|:-------|:------------------------------------------------------------|:---------------------------------------|
| 2.3.1  | [Page views](#pageview)                                     | 'page_view'                            |
| 2.3.2  | [Page pings](#pagepings)                                     | 'page_ping'                            |
| 2.3.3  | [Ecommerce transactions](#ecomm)                            | 'transaction' and 'transaction_item'   |
| 2.3.4  | [Errors](#error)                                            | 'error'                                |
| 2.3.5  | [Custom structured events](#customstruct)                   | 'struct'                               |
| 2.3.6  | [Custom unstructured events](#customunstruct)               | 'unstruct'                             |

Details of which fields are available for which events are given below:

<a name="pageview" />
#### 2.3.1 Page views

There are currently no fields that are specific to `page_view` events: all the fields that are required are part of the standard fields available for any [web-based event](#web) e.g. `page_urlscheme`, `page_title`.

Back to [top](#top).

<a name="pagepings" />
#### 2.3.2 Page pings

There are four additional fields included with page pings that indicate how a user has scrolled over a web page since the last page ping:

| **Field**       | **Type** | **Description** | **Reqd?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:---------------|
| `pp_xoffset_min`| integer  | Minimum page x offset seen in the last ping period | No | 0 | 
| `pp_xoffset_max`| integer  | Maximum page x offset seen in the last ping period | No | 100 | 
| `pp_yoffset_min`| integer  | Minimum page y offset seen in the last ping period | No | 0 | 
| `pp_yoffset_max`| integer  | Maximum page y offset seen in the last ping period | No | 200 | 

Back to [top](#top).

<a name="ecomm" />
#### 2.3.3 Ecommerce transactions

There are a large number of fields specifically for transaction events.

Fields that start `tr_` relate to the transaction as a whole. Fields that start `ti_` refer to the specific item included in the transaction. (E.g. a product in the basket.) Single transactions typically span multiple lines of data: there will be a single line where `event` = `transaction`, where the `tr_` fields are set, and multiple lines (one for each product included) where `event` = `transaction_item` and the `ti_` fields are set.

| **Field**           | **Type** | **Description** | **Reqd?** | **Example**    |
|:--------------------|:---------|:----------------|:----------|:---------------|
| `tr_orderid`        | text     | Order ID        | Yes       | '#134'         |
| `tr_affiliation`    | text     | Transaction affiliation (e.g. store where sale took place) | No | 'web' |
| `tr_total`          | decimal  | Total transaction value | Yes | 12.99          |
| `tr_tax`            | decimal  | Total tax included in transaction value | No | 3.00 |
| `tr_shipping`       | decimal  | Delivery cost charged | No  | 0.00           |
| `tr_total_base`*    | decimal  | Total in base currency | No | 12.99          |
| `tr_tax_base`*      | decimal  | Total tax in base currency | No | 3.00 |
| `tr_shipping_base`* | decimal  | Delivery cost in base currency | No  | 0.00           |
| `tr_city`           | text     | Delivery address, city | No | 'London'       |
| `tr_state`          | text     | Delivery address, state | No| 'Washington'   |
| `tr_country`        | text     | Delivery address, country | No| 'France'       |
| `tr_currency`       | text     | Currency        | No        | 'USD'          |
| `ti_orderid`        | text     | Order ID        | Yes       | '#134'         |
| `ti_sku`            | text     | Product SKU     | Yes       | 'pbz00123'     |
| `ti_name`           | text     | Product name    | No        | 'Cone pendulum'|
| `ti_category`       | text     | Product category| No        | 'New Age'      |
| `ti_price`          | text     | Product unit price | Yes    | 9.99           |
| `ti_price_base`*    | text     | Price in base currency | No    | 9.99           |
| `ti_quantity`       | text     | Number of product in transaction | Yes | 2         |
| `ti_currency`       | text     | Currency        | No        | 'EUR'          |
| `base_currency`*    | text     | Reporting currency | No        | 'GBP'          |

\* Set exclusively by the [[Currency conversion enrichment]].

Back to [top](#top).

<a name="error" />
#### 2.3.4 Error tracking

This has not been implemented yet.

Back to [top](#top).

<a name="customstruct" />
#### 2.3.5 Custom structured events

If you wish to track an event that Snowplow does not recognise as a first class citizen (i.e. one of the events listed above), then you can track them using the generic 'custom structured events'. There are five fields that are available to store data related to custom events:

| **Field**       | **Type** | **Description** | **Reqd?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:---------------|
| `se_category`   | text     | Category of event | Yes     | 'ecomm', 'video' |
| `se_action`     | text     | Action performed / event name | Yes | 'add-to-basket', 'play-video' |
| `se_label`      | text     | The object of the action e.g. the ID of the video played or SKU of the product added-to-basket | No | 'pbz00123' |
| `se_property`   | text     | A property associated with the object of the action | No | 'HD', 'large' |
| `se_value`      | decimal  | A value associated with the event / action e.g. the value of goods added-to-basket | No | 9.99 |


See [issue 74](https://github.com/snowplow/snowplow/issues/74) for additional information.

Back to [top](#top).

<a name="customunstruct" />
#### 2.3.6 Custom unstructured events

Custom unstructured events are a flexible tool that enable Snowplow users to define their own event types and send them into Snowplow.

When a user sends in a custom unstructured event, they do so as a JSON of name-value properties that conforms to a JSON schema defined for the event earlier. 

The unstructured event is not part of the `atomic.events` table; instead, for users running on Redshift, it is shredded into its own table. The fields in this table will be determined by the JSON schema defined for the event in advance. Users can query just the table for that particular unstructured event, if that's all that's required for their analysis, or join that table back to the `atomic.events` table by `atomic.my_example_unstructured_event_table.root_id = atomic.events.root_id`.

Back to [top](#top).

<a name="customcontexts" />
#### 2.3.7 Contexts

Contexts enable Snowplow users to define their own entities that are related to events, and fields that are related to each of those entities. For example, an online retailer may choose to define a `user` context, to store information about a particular user, which might include data points like the users Facebook ID, age, membership details etc. In addition, they may also define a `product` context, with product data e.g. SKU, name, created date, description, tags etc. 

An event can have any number of custom contexts attached. Each context is passed into Snowplow as a JSON. Additionally, the Snowplow Enrichment process can derive additional contexts.

Contexts are not part of the `atomic.events` table; instead, for users running on Redshift, Snowplow will shred each context JSON into a dedicated table in the `atomic` schema, making it much more efficient for analysts to query data passed in in any one of the contexts. Those contexts can be joined back to the core `atomic.events` table on `atomic.my_custom_context_table.root_id = atomic.events.event_id`, which is a one-to-one join.

Back to [top](#top).

<a name="specific-unstruct-events-contexts" />
### 2.4 Specific unstructured events and custom contexts

These are also a variety of unstructured events and custom contexts defined by Snowplow. You can find their schemas [here][snowplow-schemas].

Back to [top](#top).

<a name="note" />
## 3. A note about storage data formats

* Currently, Snowplow data is stored in S3 (for processing in Apache Hive, Pig, and / or Mahout), Redshift and PostgreSQL (for processing by any SQL-compatible analytics package).
* There are minor differences between the structure of data in both formats. These relate to data structures that Hive and PostgreSQL support (e.g. JSONs) that Redshift does not
* Nevertheless, the structure of both is similar: representing a fat table
* Going forwards our intention is to move the storage format for data on S3 from the current flatfile structure to Avro. **This** will become the 'canonical Snowplow data structure'. Other formats (e.g. Redshift, PostgreSQL etc.) will simply be 'flattened' versions of the same data. We have outlined some of our plans in [this blog post][avro-blog-post].

Back to [top](#top).

[shredding]: https://github.com/snowplow/snowplow/wiki/Shredding
[avro-blog-post]: http://snowplowanalytics.com/blog/2013/02/04/help-us-build-out-the-snowplow-event-model/
[json-schema]: http://json-schema.org/
[snowplow-schemas]: https://github.com/snowplow/iglu-central/tree/master/schemas/com.snowplowanalytics.snowplow
