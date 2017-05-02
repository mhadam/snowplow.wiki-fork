[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Storage**](storage-documentation) > Canonical Event Model pre-r63


This relates to the Canonical Event Model prior to Snowplow Release 63 Red-Cheeked Cordon-Bleu. For other versions:

* [Canonical Event Model](Canonical-event-model) (Latest)
* [Canonical Event Model pre-Release 73](Canonical-event-model-v72)
* [Canonical Event Model pre-Release 71](Canonical-event-model-v70)

<a name="top" />

# Canonical data structure

1. [Overview](#overview)
2. [The Snowplow canonical data structure: understanding the individual fields](#model)
3. [A note about data storage formats](#note)

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
- 2.4 [**Specific unstructured events**](#specific-unstruct)
  - 2.4.1 [Link clicks](#link-click)
  - 2.4.2 [Ad impressions](#ad-impression)
  - 2.4.3 [Ad clicks](#ad-click)
  - 2.4.4 [Ad conversions](#ad-conversion)
  - 2.4.5 [Screen views](#screen-view)
  - 2.4.6 [Social events](#social)
  - 2.4.7 [Item views](#itemview)

<a name="common" />

### 2.1 Common fields (platform and event independent)

<a name="application" />

#### 2.1.1 Application fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `app_id`        | text     | Application ID  | Yes       | Yes       | 'angry-birds'  |
| `platform`      | text     | Platform        | Yes       | Yes       | 'web'          |

The application ID is used to distinguish different applications that are being tracked by the same Snowplow stack.

The platform ID is used to distinguish the same app running on different platforms e.g. `iOS` vs `web`.

Back to [top](#top).

<a name="date-time" />

#### 2.1.2 Date / time fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `collector_tstamp`| timestamp | Time stamp for the event recorded by the collector | Yes    | Yes       | '2013-11-26 00:02:05'   |
| `dvce_tstamp`   | timestamp | Timestamp event was recorded on the client device | No | Yes | '2013-11-26 00:03:57.885' |
| `etl_tstamp`   | timestamp | Timestamp event began ETL | No | Yes | '2017-01-26 00:01:25.292' |
| `os_timezone`   | text     | Client operating system timezone | No | Yes | 'Europe/London' |

We are currently considering extending the date / time fields to store the date / time as recorded on the client and server in separate fields. See [issue 149](https://github.com/snowplow/snowplow/issues/149) for details.

Back to [top](#top).

<a name="eventtransaction" />

#### 2.1.3 Event / transaction fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `event`         | text     | The type of event recorded | Yes | Yes  | 'page_view'    |
| `event_id`      | text     | A UUID for each event | Yes | Yes       | 'c6ef3124-b53a-4b13-a233-0088f79dcbcb' |
| `txn_id`        | int      | Transaction ID set client-side, used to de-dupe records | No | Yes | 421828 |

A complete list of event types is given [here](#event).

Back to [top](#top).

<a name="version" />

#### 2.1.4 Snowplow version fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `v_tracker`     | text     | Tracker version | Yes       | Yes       | 'no-js-0.1.0'  |
| `v_collector`   | text     | Collector version | Yes     | Yes       | 'clj-tomcat-0.1.0', 'cf' |
| `v_etl`         | text     | ETL version     | Yes       | Yes       | 'serde-0.5.2'  |
| `name_tracker`  | text     | Tracker namespace     | No        | Yes       | 'cloudfront-1' |

Some Snowplow Trackers allow the user to name each specific Tracker instance. `name_tracker` corresponds to this name, and can be used to distinguish which tracker generated which events.

Back to [top](#top).

<a name="user" />

#### 2.1.5 User-related fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `user_id`       | text     | Unique ID set by business | No | Yes | 'jon.doe@email.com' |
| `domain_userid` | text     | User ID set by Snowplow using 1st party cookie | No | Yes | 'bc2e92ec6c204a14' |
| `network_userid`| text     | User ID set by Snowplow using 3rd party cookie | No | Yes | 'ecdff4d0-9175-40ac-a8bb-325c49733607' |
| `user_ipaddress` | text    | Ueser IP address | No       | Yes       | '92.231.54.234' |
| `domain_sessionidx`      | int      | A visit / session identifier | No | Yes | 3              |

`domain_sessionidx` is the number of the current user session. For example, an event occurring during a user's first session would have `domain_sessionidx` set to 1. The JavaScript Tracker calculates this field by storing a visit count in a first-party cookie. Whenever the Tracker fires an event, if more than 30 minutes have elapsed since the last event, the visitor count is increased by 1. (Whenever an event is fired, a "session cookie" is created and set to expire in 30 minutes. This is how the Tracker can tell whether the visit count should be incremented.) Thirty minutes is the default value and can be changed using the `setSessionCookieTimeout` method.


Back to [top](#top).

<a name="device" />

#### 2.1.6 Device and operating system fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `useragent`     | text     | Raw useragent   | Yes       | Yes       |                |
| `dvce_type`     | text     | Type of device  | No        | Yes       | 'Computer'     |
| `dvce_ismobile` | boolean  | Is the device mobile? | No  | Yes       | 1              |
| `dvce_screenheight` | int  | Screen height in pixels | No | Yes      | 1024           |
| `dvce_screenwidth`  | int  | Screen width in pixels  | No |          | 1900           |
| `os_name`       | text     | Name of operating system | No | Yes     | 'Android'      |
| `os_family`     | text     | Operating system family | No | Yes      | 'Linux'        |
| `os_manufacturer` | text   | Company responsible for OS | No | Yes   | 'Apple'        |

Back to [top](#top).

<a name="location" />

#### 2.1.7 Location fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `geo_country`   | text     | ISO 3166-1 code for the country the visitor is located in | No  | Yes | 'GB', 'US' |
| `geo_region`    | text     | ISO-3166-2 code for country region the visitor is in | No | Yes | 'I9', 'TX' |
| `geo_city`      | text     | City the visitor is in | No | Yes        | 'New York', 'London'       |
| `geo_zipcode`  | text     | Postcode the visitor is in | No | Yes    | '94109'           |
| `geo_latitude`  | text     | Visitor location latitude | No | Yes     | 37.443604      |
| `geo_longitude` | text     | Visitor location longitude | No | Yes    | -122.4124      |
| `geo_region_name` | text     | Visitor region name | No | Yes    | 'Florida'      |

<a name="ip" />

#### 2.1.7 IP address-based fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `ip_isp`   | text     | Visitor's ISP | No  | Yes | 'FDN Communications' |
| `ip_organization`    | text     | Organization associated with the visitor's IP address - defaults to ISP name if none is found | No | Yes | 'Bouygues Telecom' |
| `ip_domain`      | text     | Second level domain name associated with the visitor's IP address | No | Yes        | 'nuvox.net'       |
| `ip_netspeed`  | text     | Visitor's connection type | No | Yes    | 'Cable/DSL'           |

<a name="platform" />

### 2.2 Platform-specific fields

Currently the only platform supported is `web`. However, as we build trackers for more platforms (e.g. iOS, Windows 8) we would expect to add platforms that are specific to each of these platforms.

<a name="web" />

#### 2.2.1 Web-specific fields

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| **Page fields** |          |                 |           |           |                |
| `page_url'      | text     | The page URL    | Yes       | Yes       | 'http://www.example.com'|
| `page_urlscheme`| text     | Scheme aka protocol | Yes   | Yes       | 'https'        |
| `page_urlhost`  | text     | Host aka domain | Yes       | yes       | '“www.snowplowanalytics.com' |
| `page_urlport`  | int      | Port if specified, 80 if not| Yes       | 80             |
| `page_urlpath`  | text     | Path to page    | No        | Yes       | '/product/index.html' |
| `page_urlquery` | text     | Querystring     | No        | Yes       | 'id=GTM-DLRG'  |
| `page_urlfragment` | text  | Fragment aka anchor | No    | Yes       | '4-conclusion' |
| `page_referrer` | text     | URL of the referrer | No    | Yes       | 'http://www.referrer.com' |
| `page_title`    | text     | Web page title  | No        | Yes       | 'Using ChartIO to visualize and interrogate Snowplow data - Snowplow Analytics' |
| `refr_urlscheme`| text     | Referer scheme  | No        | Yes       | 'http'         |
| `refr_urlhost`  | text     | Referer host    | No        | Yes       | 'www.bing.com' |
| `refr_urlport`  | int      | Referer port    | No        | Yes       | 80 |
| `refr_urlpath`  | text     | Referer page path | No      | Yes       | '/images/search' |
| `refr_urlquery` | text     | Referer URL querystring | No | Yes      | 'q=psychic+oracle+cards' |
| `refr_urlfragment` | text   | Referer URL fragment | No   | Yes       |                |
| `refr_medium`   | text     | Type of referer | No | Yes | 'search', 'internal' |
| `refr_source`   | text     | Name of referer if recognised | No | Yes | 'Bing images' |
| `refr_term`     | text     | Keywords if source is a search engine | No | Yes | 'psychic oracle cards' |
| **Document fields** |      |                 |           |           |                |
| `doc_charset`   | text     | The page’s character encoding | No | Yes | , 'UTF-8' |
| `doc_width`     | int      | The page's width in pixels  | No | Yes  | 1024       |
| `doc_height`    | int      | The page's height in pixels | No | Yes  | 3000       |
| **Marketing / traffic source fields** |          |                 |           |           |                |
| `mkt_medium`    | text     | Type of traffic source | No | Yes       | 'cpc', 'affiliate', 'organic', 'social' |
| `mkt_source`    | text     | The company / website where the traffic came from | No | Yes | 'Google', 'Facebook' |
| `mkt_term`      | text     | Any keywords associated with the referrer | Yes | No | 'new age tarot decks' |
| `mkt_content`   | text     | The content of the ad. (Or an ID so that it can be looked up.) | No | Yes | 13894723 |
| `mkt_campaign`  | text     | The campaign ID | No        | Yes        | 'diageo-123' |
| **Browser fields** |          |                 |           |           |                |
| `user_fingerprint` | int   | A user fingerprint generated by looking at the individual browser features | No | No | 2161814971 |
| `connection_type` | text   | Type of internet connection | No | No   | - |
| `cookie`        | boolean  | Does the browser support persistent cookies? | No | Yes | 1 |
| `br_name`       | text     | Browser name  | No | Yes   | 'Firefox 12' |
| `br_version`    | text     | Browser version  | No | Yes   | '12.0' |
| `br_family`     | text     | Browser family  | No | Yes   | 'Firefox' |
| `br_type`       | text     | Browser type  | No | Yes   | 'Browser' |
| `br_renderengine` | text   | Browser rendering engine  | No | Yes   | 'GECKO' |
| `br_lang`       | text     | Language the browser is set to  | No | Yes   | 'en-GB' |
| `br_features_pdf` | boolean     | Whether the browser recognizes PDFs | No | Yes   | 1 |
| `br_features_flash` | boolean     | Whether Flash is installed | No | Yes   | 1 |
| `br_features_java` | boolean     | Whether Java is installed | No | Yes   | 1 |
| `br_features_director` | boolean     | Whether Adobe Shockwave is installed | No | Yes   | 1 |
| `br_features_quicktime` | boolean     | Whether QuickTime is installed | No | Yes   | 1 |
| `br_features_realplayer` | boolean     | Whether RealPlayer is installed | No | Yes   | 1 |
| `br_features_windowsmedia` | boolean     | Whether mplayer2 is installed | No | Yes   | 1 |
| `br_features_gears` | boolean     | Whether Google Gears is installed | No | Yes   | 1 |
| `br_features_silverlight` | boolean     | Whether Microsoft Silverlight is installed | No | Yes   | 1 |
| `br_cookies` | boolean     | Whether cookies are enabled | No | Yes   | 1 |
| `br_colordepth` | int      | Bit depth of the browser color palette  | No | Yes | 24 |
| `br_jsversion`  | text     | Javascript version | No     | No        | - |
| `br_viewheight`| int     | Viewport height    | No     | Yes         | 1000 |
| `br_viewwidth` | int     | Viewport width     | No     | Yes         | 1000 |

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

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `pp_xoffset_min`| integer  | Minimum page x offset seen in the last ping period | No | Yes | 0 |
| `pp_xoffset_max`| integer  | Maximum page x offset seen in the last ping period | No | Yes | 100 |
| `pp_yoffset_min`| integer  | Minimum page y offset seen in the last ping period | No | Yes | 0 |
| `pp_yoffset_max`| integer  | Maximum page y offset seen in the last ping period | No | Yes | 200 |

Back to [top](#top).

<a name="ecomm" />

#### 2.3.3 Ecommerce transactions

There are a large number of fields specifically for transaction events.

Fields that start `tr_` relate to the transaction as a whole. Fields that start `ti_` refer to the specific item included in the transaction. (E.g. a product in the basket.) Single transactions typically span multiple lines of data: there will be a single line where `event` = `transaction`, where the `tr_` fields are set, and multiple lines (one for each product included) where `event` = `transaction_item` and the `ti_` fields are set.

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `tr_orderid`    | text     | Order ID        | Yes       | Yes       | '#134'         |
| `tr_affiliation`| text     | Transaction affiliation (e.g. store where sale took place) | No | Yes | 'web' |
| `tr_total`      | decimal  | Total transaction value | Yes | Yes     | 12.99          |
| `tr_tax`        | decimal  | Total tax included in transaction value | No | Yes | 3.00 |
| `tr_shipping`   | decimal  | Delivery cost charged | No  | Yes       | 0.00           |
| `tr_city`       | text     | Delivery address, city | No | Yes       | 'London'       |
| `tr_state`      | text     | Delivery address, state | No| Yes       | 'Washington'   |
| `tr_country`    | text     | Delivery address, country | No| Yes     | 'France'       |
| `ti_orderid`    | text     | Order ID        | Yes       | Yes       | '#134'         |
| `ti_sku`        | text     | Product SKU     | Yes       | Yes       | 'pbz00123'     |
| `ti_name`       | text     | Product name    | No        | Yes       | 'Cone pendulum'|
| `ti_category`   | text     | Product category| No        | Yes       | 'New Age'      |
| `ti_price`      | text     | Product unit price | Yes    | Yes       | 9.99           |
| `ti_quantity`   | text     | Number of product in transaction | Yes | Yes | 2         |

Back to [top](#top).

<a name="error" />

#### 2.3.4 Error tracking

This has not been implemented yet.

Back to [top](#top).

<a name="customstruct" />

#### 2.3.5 Custom structured events

If you wish to track an event that Snowplow does not recognise as a first class citizen (i.e. one of the events listed above), then you can track them using the generic 'custom structured events'. Currently there are five fields that are available to store data related to custom events: we plant to increase this to 25 in the near future:

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `se_category`   | text     | Category of event | Yes     | Yes       | 'ecomm', 'video' |
| `se_action`     | text     | Action performed / event name | Yes | Yes | 'add-to-basket', 'play-video' |
| `se_label`      | text     | The object of the action e.g. the ID of the video played or SKU of the product added-to-basket | No | Yes | 'pbz00123' |
| `se_property`   | text     | A property associated with the object of the action | No | Yes | 'HD', 'large' |
| `se_value`      | decimal  | A value associated with the event / action e.g. the value of goods added-to-basket | No | Yes | 9.99 |


See [issue 74](https://github.com/snowplow/snowplow/issues/74) for additional information.

Back to [top](#top).

<a name="customunstruct" />

#### 2.3.6 Custom unstructured events

Custom unstructured events are a flexible tool that enable Snowplow users to define their own event types and send them into Snowplow.

When a user sends in a custom unstructured event, they do so as a JSON of name-value properties, that conforms to a JSON schema defined for the event earlier.

The complete JSON envelop for the event is loaded into the `atomic.events` table, in the `unstruct_event` field as shown below:

| **Field**        | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:-----------------|:---------|:----------------|:----------|:----------|:---------------|
| `unstruct_event` | JSON     | Name of the event | Yes     | Yes       | {"schema":"iglu:com.mycompany/product_view/jsonschema/1-0-1", "data":{"sku":"14424"}} |

In addition, for Snowplow users running Redshift, the unstructured event is shredded into its own table in Amazon Redshift. The fields in this table will be determined by the JSON schema defined for the event in advance. Users can query just the table for that particular unstructured event, if that's all that's required for their analysis, or join that table back to the `atomic.events` table by `atomic.my_example_unstructured_event_table.root_id = atomic.events.root_id`.

Back to [top](#top).

<a name="customcontexts" />

#### 2.3.7 Custom contexts

Custom contexts enable Snowplow users to define their own entities that are related to events, and fields that are related to each of those entities. For example, an online retailer may choose to define a `user` context, to store information about a particular user, which might include data points like the users Facebook ID, age, membership details etc. In addition, they may also define a `product` context, with product data e.g. SKU, name, created date, description, tags etc.

An event can have any number of custom contexts attached. Each context is passed into Snowplow as a JSON.

The array of contexts JSONs related to an event is loaded directly into the `atomic.events` table in the `contexts` field, as described below:

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `contexts`   | array     | Contexts attached to event | Yes     | Yes       | [{"schema":"iglu:com.acme/page_type/jsonschema/1-0-0", "data":{"type":"test"}}] |

In addition, for users running on Redshift, Snowplow will shred each context JSON into a dedicated table in the `atomic` schema, making it much more efficient for analysts to query data passed in in any one of the contexts. Those contexts can be joined back to the core `atomic.events` table on `atomic.my_custom_context_table.root_id = atomic.events.event_id`, which is a one-to-one join.

Back to [top](#top).

<a name="specific-unstruct" />

### 2.4 Specific unstructured events

These are unstructured events defined by Snowplow. Their schemas can be found [here][snowplow-schemas].

<a name="link-click" />

#### 2.4.1 Link clicks

| **Field**        | **Type** | **Description**     | **Reqd?** | **Example**    |
|:-----------------|:---------|:--------------------|:----------|:---------------|
| `targetUrl`      | text     | Link href attribute | Yes       | 'http://www.example.com' |
| `elementId`      | text     | ID of the link element | No     | 'firstLink'         |
| `elementClasses` | array    | CSS classes of the link element | No | ['title', 'foreground'] |
| `elementTarget`  | text     | Target of the link element  | No | '_blank' |

<a name="ad-impression" />

#### 2.4.2 Ad impressions

| **Name**       | **Type** | **Description**                            | **Requd?**
|---------------:|:---------|:-------------------------------------------|:---------------------|
| `impressionId` | text     | Identifier for the particular impression instance   | No |
|    `costModel` | text     | The cost model for the campaign: 'cpc', 'cpm', or 'cpa'  | No |
|         `cost` | decimal  | Ad cost   |    No    |
|   `targetUrl`  | text     | The destination URL  |        No   |
|     `bannerId` | text     | Adserver identifier for the ad banner (creative) being displayed  | No  |
|       `zoneId` | text     | Adserver identifier for the zone where the ad banner is located | No  |
| `advertiserId` | text     | Adserver identifier for the advertiser which the campaign belongs to      | No   |
|   `campaignId` | text     | Adserver identifier for the ad campaign which the banner belongs to    |  No |

<a name="ad-click" />

#### 2.4.3 Ad clicks

| **Name**       | **Type** | **Description**                            | **Reqd?**
|---------------:|:---------|:-------------------------------------------|:-----------
|   `targetUrl`  | text     | The destination URL  |        yes   |
|      `clickId` | text     | Identifier for the particular click instance   | No  |
|    `costModel` | text     | The cost model for the campaign: 'cpc', 'cpm', or 'cpa' |  No  |
|         `cost` | decimal  | Ad cost   |    No    |
|     `bannerId` | text     | Adserver identifier for the ad banner (creative) being displayed  |  No
|       `zoneId` | text     | Adserver identifier for the zone where the ad banner is located  | No
| `advertiserId` | text     | Adserver identifier for the advertiser which the campaign belongs to   |  No |
|   `campaignId` | text     | Adserver identifier for the ad campaign which the banner belongs to   |No |

<a name="ad-conversion" />

#### 2.4.4 Ad conversions

| **Name**       | **Type** | **Description**                            | **Reqd?**             |
|---------------:|:--------------|:-------------------------------------------|:--------------------
| `conversionId` | text            | Identifier for the particular conversion instance   | No
|    `costModel` | text            | The cost model for the campaign: 'cpc', 'cpm', or 'cpa'   |  No
|         `cost` | decimal            | Ad cost   |    No    |
|     `category` | text            | Conversion category                        |    No                              |
|       `action` | text            | The type of user interaction, e.g. 'purchase' | No                           |
|     `property` | text            | Describes the object of the conversion        | No                           |
| `initialValue` | decimal            | How much the conversion is initially worth   | number                           |
| `advertiserId` | text            | Adserver identifier for the advertiser which the campaign belongs to |  No   |
|   `campaignId` | text            | Adserver identifier for the ad campaign which the banner belongs to  |  No  |

<a name="screen-view" />

#### 2.4.5 Screen views

| **Name**       | **Type**  | **Description** | **Reqd?**    |
|---------------:|:----------|:----------------|:-------------|
| `name`         | text      | Screen name     | No           |
|   `id`         | text      | Screen ID       | No           |

<a name="social" />

#### 2.4.6 Social events

This has not been developed yet. However, the intention is to build support for the following fields for use recording social events (e.g. *likes*, *+1s*).

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `social_action` | text     | Social action performed | Yes | No      | 'like'         |
| `social_network`| text     | Social network  | Yes       | No        | 'Google+'      |
| `social_target` | text     | The object of the social action e.g. the music track liked, the tweet retweeted | No | No | 'pbz00123' |
| `social_pagepath` | text   | The URL of the page where the action occurred. This is generally the same as `page_url`, so does not need to be set | No | No | - |

<a name="itemview" />

#### 2.4.7 Item views

This has not been implemented yet. The intention is to implement the following fields:

| **Field**       | **Type** | **Description** | **Reqd?** | **Impl?** | **Example**    |
|:----------------|:---------|:----------------|:----------|:----------|:---------------|
| `item_id`         | text     | Item ID e.g. SKU if item is a product | Yes       | No        | 'pbz00123' |
| `item_name`       | text     | Item name or title | Yes    | No        | 'Cone pendulum' |
| `item_displayformat`| text   | Type of item listing. Used to distinguish views of a product summary (in a catalogue page) vs a detailed view (on the product page). | No  | 'summary-view' |
| `item_rank`     | integer  | Item rank (position if there is a list of items displayed on the page) | No | No | 3 |
| `item_location` | text     | Location of the item on the web page | No | No | 'div-cat-4' |

For additional details see [this gist](https://gist.github.com/4327909) and [issue 113](https://github.com/snowplow/snowplow/issues/113)

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
