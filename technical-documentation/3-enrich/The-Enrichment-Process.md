[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Enrichment**](Enrichment) > The Enrichment Process

###Overview

Enrichment process parses raw Snowplow events and performs the following:

1. Extracts data
2. Validates data against [Snowplow Tracker Protocol](snowplow-tracker-protocol) and JSON schema
3. Enriches data (adds extra value derived from the tracked/captured data), so called "dimension widening"
4. Writes enriched data out

Therefore feeding in a raw Snowplow event will produce either the enriched event with additional data (context), modified (enriched) data or a `bad` record.

> **JSON schema** specifies a *JSON*-based format to define the structure of JSON data for validation, documentation, and interaction control.

> **JSON** (JavaScript Object Notation) is an open-standard format that uses human-readable text to transmit data objects consisting of attribute–value pairs.

We distinguish 3 types of enrichment:

- Hardcoded enrichments loading `atomic.events` (legacy)
- Configurable enrichments loading `atomic.events` (legacy)
- Configurable enrichments adding new contexts to the `derived_contexts` JSON array

*Legacy enrichments* are those which populate `atomic.events` table as opposed to enrichment’s dedicated tables. The hardcoded legacy enrichments normally take place as part of common enrichment process and they precede configurable enrichments.

*Configurable enrichments* are those controlled with `--enrichments` option passed to the ETL (Extract, Transform, Load) runner. They often depend on the data produced by the common enrichment process.

> **ETL** stands for **E**xtract, **T**ransform, **L**oad.

During the common enrichment process the data received from collector(s) is mapped according to our [Canonical Event Model](canonical-event-model).

The raw data undergoing "dimension widening" (enrichment) listed as per following:

- [Hardcoded enrichment](#hardcoded-enrichment)
- [Configurable enrichment](#configurable-enrichment)

<a name="hardcoded-enrichment" />

###Hardcoded enrichment

The following fields are populated depending on whether the tracker provided the corresponding value or not.

Raw Parameter | Enriched Parameter | Purpose
:---|:---|:---
`eid` | `event_id` | The unique event identifier (UUID). Assigned during enrichment if not provided with `eid`
`cv` | `v_collector` | Collector type/version
`tnuid` | `network_userid` | User ID set by Snowplow using 3rd party cookie. Overwriten with tracker-set `tnuid`.
`ip` | `user_ipaddress` | Snowplow collectors log IP address as standard. However, you can override the value derived from the collector by populating this value in the tracker.
`ua` | `useragent` | Raw useragent (browser string). Could be overwritten with `ua`.

The following fields are populated depending on the collector and ETL (Extract, Transform, Load) utilized in the pipeline.

Added Parameter | Purpose
:---|:---
`v_etl` | Host ETL version
`etl_tstamp` | Timestamp event began ETL
`collector_tstamp` | Time stamp for the event recorded by the collector

The raw parameter `res` (if present) representing the screen/monitor resolution and coming in as a combination of width and height (ex. `1280x1024`) is broken up into separate entities.

Added Parameter | Purpose
:---|:---
`dvce_screenwidth` | Screen / monitor width
`dvce_screenheight` | Screen / monitor height

The `url` parameter provides the value for `page_url` in `atomic.events`, which represents the current page's URL. The following parts are extracted and populate separate fields as outlined below.

Added Parameter | Purpose
:---|:---
`page_urlscheme` | Scheme (protocol), ex. "http"
`page_urlhost` | Host (domain), ex. "www.snowplowanalytics.com"
`page_urlport` | Port if specified, 80 if not
`page_urlpath` | Path to page, ex. "/product/index.html"
`page_urlquery` | Querystring, ex. "id=GTM-DLRG"
`page_urlfragment` | Fragment (anchor), ex. "4-conclusion"

Similarly, `page_referrer` gets the value from `refr`, which represents the referer’s URL, and the following parts are extracted and populate separate fields as shown below.

Added Parameter | Purpose
:---|:---
`refr_urlscheme` | Scheme (protocol)
`refr_urlhost` | Host (domain)
`refr_urlport` | Port if specified, 80 if not
`refr_urlpath` | Path to page
`refr_urlquery` | Querystring
`refr_urlfragment` | Fragment (anchor)

Additionally the derived timestamp is calculated, `derived_tstamp`. See [this blog post](http://snowplowanalytics.com/blog/2015/09/15/improving-snowplows-understanding-of-time/) for more details.

Finally, contexts, unstructured events and the relevant configurable enrichments (if enabled) are validated against their corresponding JSON schemas and the array of the derived contexts is assembled.

<a name="configurable-enrichment" />

###Configurable enrichment

The configurable enrichment are listed below. Follow the corresponding links to find out more.

The enrichments which write the data into `atomic.events` table (legacy enrichments):

- [IP anonymization enrichment](IP-anonymization-enrichment)
- [IP lookups enrichment](IP-lookups-enrichment)
- [Campaign attribution enrichment](Campaign-attribution-enrichment)
- [Currency conversion enrichment](Currency-conversion-enrichment)
- [referer-parser enrichment](referer-parser-enrichment)
- [user-agent-utils enrichment](user-agent-utils-enrichment)
- [Event fingerprint enrichment](Event-fingerprint-enrichment)

The below list contains the enrichment which create a separate context and thus are loaded into their dedicated tables (as opposed to `atomic.events`):

- [JavaScript script enrichment](JavaScript-script-enrichment)
- [ua-parser enrichment](ua-parser-enrichment)
- [Cookie extractor enrichment](Cookie-extractor-enrichment)
- [Weather enrichment](Weather-enrichment)
