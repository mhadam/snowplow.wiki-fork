[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Enrichment**](Enrichment) > [[Stream Enrich]]

## Overview

Stream Enrich is an [Amazon Kinesis][kinesis] app, written in Scala and using the Kinesis Client Library, which:

1. **Reads** raw Snowplow events off a Kinesis stream populated by the Scala Stream Collector
2. **Validates** each raw event
2. **Enriches** each event (e.g. infers the location of the user from his/her IP address)
3. **Writes** the enriched Snowplow event to another Kinesis stream

It is designed to be used downstream of the [[Scala Stream Collector]].

It also supports reading raw events from `stdin` and writing enriched events to `stdout`, which is useful for debugging.

Stream Enrich utilizes the [scala-common-enrich][common-enrich] Scala project to enrich events and the [SnowplowRawEvent][schema] for reading Thrift-serialized objects collected with the Scala Stream Collector.

## Steam Enrich output

The result of the enrichment process is a TSV representation of the event breakdown of which is outlined below. For the description of each field, please, refer to the [Canonical Event Model](canonical-event-model).

<br>

**The application (site, game, app, etc.) this event belongs to, and the tracker platform**

----------

- `app_id`: String
- `platform`: String

<br>

**Date/time**

----------

- `etl_tstamp`: String
- `collector_tstamp`: String
- `dvce_created_tstamp`: String

<br>

**Transaction (i.e. this logging event)**

----------

- `event`: String
- `event_id`: String
- `txn_id`: String

<br>

**Versioning**

----------

- `name_tracker`: String
- `v_tracker`: String
- `v_collector`: String
- `v_etl`: String

<br>

**User and visit**

----------

- `user_id`: String
- `user_ipaddress`: String
- `user_fingerprint`: String
- `domain_userid`: String
- `domain_sessionidx`: Integer
- `network_userid`: String

<br>

**Location**

----------

- `geo_country`: String
- `geo_region`: String
- `geo_city`: String
- `geo_zipcode`: String
- `geo_latitude`: Float
- `geo_longitude`: Float
- `geo_region_name`: String

<br>

**Other IP lookups**

----------

- `ip_isp`: String
- `ip_organization`: String
- `ip_domain`: String
- `ip_netspeed`: String

<br>

**Page**

----------

- `page_url`: String
- `page_title`: String
- `page_referrer`: String

<br>

**Page URL components**

----------

- `page_urlscheme`: String
- `page_urlhost`: String
- `page_urlport`: Integer
- `page_urlpath`: String
- `page_urlquery`: String
- `page_urlfragment`: String

<br>

**Referrer URL components**

----------

- `refr_urlscheme`: String
- `refr_urlhost`: String
- `refr_urlport`: Integer
- `refr_urlpath`: String
- `refr_urlquery`: String
- `refr_urlfragment`: String

<br>

**Referrer details**

----------

- `refr_medium`: String
- `refr_source`: String
- `refr_term`: String

<br>

**Marketing**

----------

- `mkt_medium`: String
- `mkt_source`: String
- `mkt_term`: String
- `mkt_content`: String
- `mkt_campaign`: String

<br>

**Custom Contexts**

----------

- `contexts`: String

<br>

**Structured Event**

----------

- `se_category`: String
- `se_action`: String
- `se_label`: String
- `se_property`: String
- `se_value`: String

<br>

**Unstructured Event**

----------

- `unstruct_event`: String

<br>

**Ecommerce transaction (from querystring)**

----------

- `tr_orderid`: String
- `tr_affiliation`: String
- `tr_total`: String
- `tr_tax`: String
- `tr_shipping`: String
- `tr_city`: String
- `tr_state`: String
- `tr_country`: String

<br>

**Ecommerce transaction item (from querystring)**

----------

- `ti_orderid`: String
- `ti_sku`: String
- `ti_name`: String
- `ti_category`: String
- `ti_price`: String
- `ti_quantity`: String

<br>

**Page Pings**

----------

- `pp_xoffset_min`: Integer
- `pp_xoffset_max`: Integer
- `pp_yoffset_min`: Integer
- `pp_yoffset_max`: Integer

<br>

**User Agent**

----------

- `useragent`: String

<br>

**Browser (from user-agent)**

----------

- `br_name`: String
- `br_family`: String
- `br_version`: String
- `br_type`: String
- `br_renderengine`: String

<br>

**Browser (from querystring)**

----------

- `br_lang`: String
- `br_features_pdf`: Byte_
- `br_features_flash`: Byte
- `br_features_java`: Byte
- `br_features_director`: Byte
- `br_features_quicktime`: Byte
- `br_features_realplayer`: Byte
- `br_features_windowsmedia`: Byte
- `br_features_gears`: Byte
- `br_features_silverlight`: Byte
- `br_cookies`: Byte
- `br_colordepth`: String
- `br_viewwidth`: Integer
- `br_viewheight`: Integer

<br>

**OS (from user-agent)**

----------

- `os_name`: String
- `os_family`: String
- `os_manufacturer`: String
- `os_timezone`: String

<br>

**Device/Hardware (from user-agent)**

----------

- `dvce_type`: String
- `dvce_ismobile`: Byte

<br>

**Device (from querystring)**

----------

- `dvce_screenwidth`: Integer
- `dvce_screenheight`: Integer

<br>

**Document**

----------

- `doc_charset`: String
- `doc_width`: Integer
- `doc_height`: Integer

<br>

**Currency**

----------

- `tr_currency`: String
- `tr_total_base`: String
- `tr_tax_base`: String
- `tr_shipping_base`: String
- `ti_currency`: String
- `ti_price_base`: String
- `base_currency`: String

<br>

Geolocation

----------

- `geo_timezone`: String

<br>

**Click ID**

----------

- `mkt_clickid`: String
- `mkt_network`: String

<br>

**ETL tags**

----------

- `etl_tags`: String


<br>

**Time event was sent**

----------

- `dvce_sent_tstamp`: String


<br>

**Referer**

----------

- `refr_domain_userid`: String
- `refr_dvce_tstamp`: String


<br>

**Derived contexts**

----------

- `derived_contexts`: String


<br>

**Session ID**

----------

- `domain_sessionid`: String


<br>

**Derived timestamp**

----------

- `derived_tstamp`: String

<br>

**Derived event vendor/name/format/version**

----------

- `event_vendor`: String
- `event_name`: String
- `event_format`: String
- `event_version`: String

<br>

**Event fingerprint**

----------

- `event_fingerprint`: String

<br>

**True timestamp**

----------

- `true_tstamp`: String


# See also:

+ [Setup guide][setup]
+ [Repository][stream-enrich]

[kinesis]: http://aws.amazon.com/kinesis/

[common-enrich]: https://github.com/snowplow/snowplow/tree/master/3-enrich/scala-common-enrich
[schema]: https://github.com/snowplow/snowplow/blob/master/2-collectors/thrift-schemas/snowplow-raw-event/src/main/thrift/snowplow-raw-event.thrift

[setup]: https://github.com/snowplow/snowplow/wiki/setting-up-stream-enrich
[stream-enrich]: https://github.com/snowplow/snowplow/tree/master/3-enrich/stream-enrich
