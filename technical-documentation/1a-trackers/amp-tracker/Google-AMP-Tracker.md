<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Trackers**](trackers) » Google AMP Tracker

## Contents

- 1. [Overview](#overview)
- 2. [Standard variables](#vars)
  - 2.1 [collectorHost](#collectorHost)
  - 2.2 [appId](#appId)
- 3. [Tracking events](#events)
  - 2.1 [Page view](#pageView)
  - 2.2 [Structured event](#structEvent)
- 4. [Analytics](#analytics)

<a name="overview" />

## 1. Overview

Snowplow is collaborating with Google on their [Accelerated Mobile Pages Project][amp](AMPP or AMP for short). AMP is an open source (Apache License 2.0) initiative to improve the mobile web experience by optimizing web pages for mobile devices.

To learn more about analytics for AMP pages see the [amp-analytics][amp-analytics] reference. For general information about AMP see [What Is AMP?][amp-what] on the [Accelerated Mobile Pages (AMP) Project][amp] site.

Snowplow is natively integrated into the project, so pages optimized with AMP HTML can be tracked in Snowplow by adding the appropriate `amp-analytics` tag to your pages:

```html
<body>
  ...
  <amp-analytics type="snowplow" id="snowplow1">
  <script type="application/json">
  {
    ...
  }
  </script>
  </amp-analytics>
</body>
```

[Back to top](#top)

<a name="vars" />

## 2. Standard variables

Certain parameters must be provided in the `"vars"` section of the tag:

```javascript
"vars": {
  ...
},
```

[Back to top](#top)

<a name="collectorHost" />

### 2.1 `collectorHost`

Specify the host to your collector like so:

```javascript
"vars": {
  "collectorHost": "snowplow-collector.acme.com",
  ...
```

Notes:

* Do *not* include the protocol aka schema (`http(s)://`)
* Do *not* include a trailing slash
* Use of HTTPS is mandatory in AMP, so your Snowplow collector **must** support HTTPS

[Back to top](#top)

<a name="appId" />

### 2.2 `appId`

You must set the application ID for the website you are tracking via AMP:

```javascript
"vars": {
  "appId": "campaign-microsite",
  ...
```

Notes:

* You do not have to use the `appId` to distinguish AMP traffic from other web traffic (unless you want to) - see the [Analytics](#analytics) section for an alternative approach

[Back to top](#top)

<a name="events" />

## 3. Tracking events

The following trigger request values are supported for the Snowplow Analytics configuration:

 * `pageView` for page view tracking
 * `structEvent` for structured event tracking

All event tracking is disabled by default; you can enable it on an event-by-event basis as follows:

[Back to top](#top)

<a name="pageView" />

### 3.1 Page view

Enable the page view tracking like so:

```html
<amp-analytics type="snowplow" id="snowplow2">
<script type="application/json">
{
  "vars": {
    "collectorHost": "snowplow-collector.acme.com",  // Replace with your collector host
    "appId": "campaign-microsite"                    // Replace with your app ID
  },
  "triggers": {
    "trackPageview": {  // Trigger names can be any string. trackPageview is not a required name
      "on": "visible",
      "request": "pageView"
    }
  }
}
</script>
</amp-analytics>
```

[Back to top](#top)

<a name="structEvent" />

### 3.2 Structured events

Structured events are user interactions with content that can be tracked independently from a web page or a screen load. "Structured" refers to the Google Analytics-style structure of having up to five fields (with only the first two required).

Events can be sent by setting the AMP trigger request value to event and setting the required event category and action fields.

The following example uses the selector attribute of the trigger to send an event when a particular element is clicked:

```html
<amp-analytics type="googleanalytics" id="snowplow3">
<script type="application/json">
{
  "vars": {
    "collectorHost": "snowplow-collector.acme.com",  // Replace with your collector host
    "appId": "campaign-microsite"                    // Replace with your app ID
  },
  "triggers": {
    "trackClickOnHeader" : {
      "on": "click",
      "selector": "#header",
      "request": "structEvent",
      "vars": {
        "structEventCategory": "ui-components",
        "structEventAction": "header-click"
      }
    }
  }
}
</script>
</amp-analytics>
```

You can set key-value pairs for the following event fields in the vars attribute of the trigger:

| **Argument**          | **Description**                                                  | **Required?** | **Validation**           |
|----------------------:|:-----------------------------------------------------------------|:--------------|:-------------------------|
| `structEventCategory` | The grouping of structured events which this `action` belongs to | Yes           | String                   |
| `structEventAction`   | Defines the type of user interaction which this event involves   | Yes           | String                   |
| `structEventLabel`    | A string to provide additional dimensions to the event data      | No            | String                   |
| `structEventProperty` | A string describing the object or the action performed on it     | No            | String                   |
| `structEventValue`    | A value to provide numerical data about the event                | No            | Int or Float             |

[Back to top](#top)

<a name="analytics" />

### 4. Analytics

All events sent via this tracker will have:

* `v_tracker` set to `amp-0.1`
* `platform` set to `web`

If you want to analyze events sent via this tracker, you may prefer to query for `v_tracker LIKE 'amp-%'` to future-proof your query against future releases of this tracker (which may change the version number).

[Back to top](#top)

[amp]: https://www.ampproject.org/
[amp-what]: https://www.ampproject.org/docs/get_started/about-amp.html
[amp-analytics]: https://www.ampproject.org/docs/reference/extended/amp-analytics.html
