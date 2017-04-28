<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Trackers**](trackers) > [**Scala Tracker**](Scala-Tracker)

**This page refers to version 0.3.0 of the Snowplow Scala Tracker. Documentation for other versions is available:**

*[Version 0.1][scala-0.1]*
*[Version 0.2][scala-0.2]*

## Contents

- 1 [Overview](#overview)
- 2 [Initialization](#init)
  - 2.1 [Tracker](#tracker-init)
  - 2.2 [Subject](#subject)
  - 2.3 [EC2 Context](#ec2)
- 3 [Sending events](#events)
  - 3.1 [`trackSelfDescribingEvent`](#trackSelfDescribingEvent)
  - 3.2 [`trackStructEvent`](#trackStructEvent)
  - 3.3 [`trackPageView`](#trackPageView)
  - 3.4 [Setting timestamp](#timestamp)
- 4 [Subject methods](#subject)

<a name="overview" />

## 1. Overview

The Snowplow Scala Tracker allows you to track Snowplow events in your Scala apps and servers.

The tracker should be straightforward to use if you are comfortable with Scala development; any prior experience with Snowplow's [[Python Tracker]], [[JavaScript Tracker]], [[Android and Java Tracker]], Google Analytics or Mixpanel (which have similar APIs to Snowplow) is helpful but not necessary.

There are three main classes which the Scala Tracker uses: subjects, emitters, and trackers.

A subject represents a single user whose events are tracked, and holds data specific to that user.

A tracker always has one active subject at a time associated with it. The default subject only has "platform=server" configured, but you can replace it with a subject containing more data. The tracker constructs events with that subject and sends them to one or more emitters, which sends them on to a Snowplow collector.

<a name="init" />

## 2. Initialization

<a name="tracker-init" />

### 2.1 Tracker

Assuming you have completed the [[Scala Tracker Setup]], you are ready to initialize the Scala Tracker.

```scala
import com.snowplowanalytics.snowplow.scalatracker._
import com.snowplowanalytics.snowplow.scalatracker.emitters._

val emitter1 = AsyncEmitter.createAndStart("mycollector.com")
val emitter2 = new SyncEmitter("myothercollector.com", port = 8080)
val emitter3 = AsyncBatchEmitter.createAndStart(host = "myothercollector.com", port = 8080, bufferSize = 32)
val tracker = new Tracker(List(emitter1, emitter2, emitter3), "mytrackername", "myapplicationid")
```

The above code:

* creates a non-blocking emitter, `emitter1`, which sends events to "mycollector.com" on the default port, port 80
* creates a blocking emitter, `emitter2`, which sends events to "myothercollector.com" on port 8080
* creates a non-blocking batch `emitter3`, which will buffer events until buffer size reach 32 events and then send all of them at once in POST request
* creates a tracker which can be used to send events to all emitters

<a name="subject" />

### 2.2 Subject

You can configure a subject with extra data and attach it to the tracker so that the data will be attached to every event:

```scala
val subject = new Subject()
  .setUserId("user-00035")
  .setPlatform(Desktop)
tracker.setSubject(subject)
```

<a name="ec2" />

### 2.3 EC2 Context

Amazon [Elastic Cloud][ec2] can provide basic information about instance running your app.
You can add this informational as additional custom context to all sent events by enabling it in Tracker after initializaiton of your tracker:

```scala
tracker.enableEc2Context()
```

<a name="events" />

## 3. Sending events

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your websites and apps.
We are constantly growing the range of functions available in order to capture that data more richly.

Tracking methods supported by the Scala Tracker at a glance:

+ [trackSelfDescribingEvent](#trackSelfDescribingEvent)
+ [trackStructEvent](#trackStructEvent)
+ [trackPageView](#trackPageView)

<a name="trackSelfDescribingEvent" />

### 3.1 `trackSelfDescribingEvent`

Use `trackSelfDescribingEvent` to track a custom Self-describing events (previously known as Unstructured Events) which consists of a name and an unstructured set of properties. This is useful when:

* You want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow), or
* You want to track events which have unpredictable or frequently changing properties

You can use its alias `trackUnstructEvent`.

| **Argument** | **Description**                                                         | **Required?** | **Type**                   |
|--------------------:|:---------------------------------------------------------------  |:--------------|:---------------------------|
| `unstructuredEvent` | Self-describing JSON containing unstructured event               | Yes           | `SelfDescribingJson`       |
| `contexts`          | List of custom contexts for the event                            | No            | `List[SelfDescribingJson]` |
| `timestamp`         | Device created timestamp or true timestamp                       | No            | `Option[Timestamp]`        |

Create a Snowplow unstructured event [self-describing JSON][self-describing-jsons] using the [json4s DSL][json4s-dsl]:

```scala
import org.json4s.JsonDSL._

val productViewEvent = SelfDescribingJson(
  "iglu:com.acme/product_view/jsonschema/1-0-0",
  ("userType" -> "tester") ~ ("sku" -> "0000013")
)
```

Send it using the `trackSelfDescribingEvent` tracker method:

```scala
tracker.trackSelfDescribingEvent(productViewEvent)
```

You can attach any number of custom contexts to an event:

```scala
val pageTypeContext = SelfDescribingJson(
  "iglu:com.acme/page_type/jsonschema/1-0-0",
  ("type" -> "promotional") ~ ("backgroundColor" -> "red")
)

val userContext = SelfDescribingJson(
  "iglu:com.acme/user/jsonschema/1-0-0",
  ("userType" -> "tester")
)

t.trackSelfDescribingEvent(productViewEvent, List(pageTypeContext, userContext))
```

<a name="trackStructEvent" />

### 3.2 `trackStructEvent`

Use `trackStructEvent` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required).

| **Argument** | **Description**                                                  | **Required?** | **Type**                   |
|-------------:|:---------------------------------------------------------------  |:--------------|:---------------------------|
| `category`   | The grouping of structured events which this `action` belongs to | Yes           | `String`                   |
| `action`     | Defines the type of user interaction which this event involves   | Yes           | `String`                   |
| `label`      | A string to provide additional dimensions to the event data      | No            | `Option[String]            |
| `property`   | A string describing the object or the action performed on it     | No            | `Option[String]            |
| `value`      | A value to provide numerical data about the event                | No            | `Option[Double]`           |
| `contexts`   | List of custom contexts for the event                            | No            | `List[SelfDescribingJson]` |
| `timestamp`  | Device created timestamp or true timestamp                       | No            | `Option[Timestamp]`        |

Example:

```scala
val pageTypeContext = SelfDescribingJson(
  "iglu:com.acme/page_type/jsonschema/1-0-0",
  ("type" -> "promotional") ~ ("backgroundColor" -> "red")
)

val userContext = SelfDescribingJson(
  "iglu:com.acme/user/jsonschema/1-0-0",
  ("userType" -> "tester")
)

t.trackStructEvent("commerce", "order", property=Some("book"), contexts=List(pageTypeContext, userContext))
```

<a name="trackPageView" />

### 3.3 `trackPageView`

Use `trackPageView` to track a user viewing a page within your app. Arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**             |
|-------------:|:------------------------------------|:--------------|:---------------------------|
| `pageUrl`    | The URL of the page                 | Yes           | `String`                   |
| `pageTitle`  | The title of the page               | No            | `Option[String]`           |
| `referrer`   | The address which linked to the page| No            | `Option[String]`           |
| `contexts`   | Custom contexts for the event       | No            | `List[SelfDescribingJson]` |
| `timestamp`  | When the pageview occurred          | No            | `Option[Timestamp]`        |

Example:

```scala
t.trackPageView("www.example.com", Some("example"), Some("www.referrer.com"))
```

<a name="timestamp" />

### 3.4 Setting timestamp

By default, Scala Tracker will generate a `dvce_created_tstamp` and add it to event payload.
You also can manually set it using `timestamp` argument in all tracking methods.
It should be in milliseconds since the Unix epoch:

```scala
tracker.trackSelfDescribingEvent(productViewEvent, Nil, Some(1432806619000L))
```

Beside of it, you can set `true_tstamp` if you have more reliable source about event timestamp.
You can tag timstamp as "true" using class `TrueTimestamp`:

```scala
tracker.trackSelfDescribingEvent(productViewEvent, Nil, Some(Tracker.TrueTimestamp(1432806619000L)))
```

Now event will be sent with `ttm` parameter instead of `dtm`.

<a name="subject" />

## 4. Subject methods

A list of the methods used to add data to the Subject class.

<a name="set-platform" />

### 4.1 Set the platform with `setPlatform`

The default platform is `Server`. These are the available alternatives, all available in the package `com.snowplowanalytics.snowplow.scalatracker`:

```scala
Server
Web
Mobile
Desktop
Tv
Console
InternetOfThings
General
```

Example usage

```scala
subject.setPlatform(Tv)
```

<a name="set-user-id" />

### 4.2 Set the user ID with `setUserId`

You can make the user ID a string of your choice:

```scala
subject.setUserId("user-000563456")
```

<a name="set-screen-resolution" />

### 4.3 Set the screen resolution with `setScreenResolution`

If your Scala code has access to the device's screen resolution, you can pass it in to Snowplow. Both numbers should be positive integers; note the order is width followed by height. Example:

```scala
subject.setScreenResolution(1366, 768)
```

<a name="set-viewport" />

### 4.4 Set the viewport dimensions with `setViewport`

Similarly, you can pass the viewport dimensions in to Snowplow. Again, both numbers should be positive integers and the order is width followed by height. Example:

```scala
subject.setViewport(300, 200)
```

<a name="set-color-depth" />

### 4.5 Set the color depth with `setColorDepth`

If your Scala code has access to the bit depth of the device's color palette for displaying images, you can pass it in to Snowplow. The number should be a positive integer, in bits per pixel.

```scala
subject.setColorDepth(24)
```

<a name="set-timezone" />

### 4.6 Setting the timezone with `setTimezone`

If your Scala code has access to the timezone of the device, you can pass it in to Snowplow:

```scala
subject.setTimezone("Europe London")
```

<a name="set-language" />

### 4.7 Setting the language with `setLang`

You can set the language field like this:

```scala
subject.setLang("en")
```

<a name="set-ip-address" />

### 4.8 Setting the IP address with `setIpAddress`

If you have access to the user's IP address, you can set it like this:

```scala
subject.setIpAddresss("34.634.11.139")
```

<a name="set-useragent" />

### 4.9 Setting the useragent with `setUseragent`

If you have access to the user's useragent (sometimes called "browser string"), you can set it like this:

```scala
subject.setUseragent("Mozilla/5.0 (Windows NT 5.1; rv:24.0) Gecko/20100101 Firefox/24.0")
```

<a name="set-domain-user-id" />

### 4.10 Setting the domain user ID with `setDomainUserId`

The `domain_userid` field of the Snowplow event model corresponds to the ID stored in the first party cookie set by the Snowplow JavaScript Tracker. If you want to match up server-side events with client-side events, you can set the domain user ID for server-side events like this:

```scala
subject.setDomainUserId("c7aadf5c60a5dff9")
```

<a name="set-network-user-id" />

### 4.11 Setting the network user ID with `setNetworkUserId`

The `network_user_id` field of the Snowplow event model corresponds to the ID stored in the third party cookie set by the Snowplow Clojure Collector and Scala Stream Collector. You can set the network user ID for server-side events like this:

```scala
subject.setNetworkUserId("ecdff4d0-9175-40ac-a8bb-325c49733607")
```

[Back to top](#top)

[scala-0.1]: https://github.com/snowplow/snowplow/wiki/Scala-Tracker-v0.1
[scala-0.2]: https://github.com/snowplow/snowplow/wiki/Scala-Tracker-v0.2

[json4s]: https://github.com/json4s/json4s
[json4s-dsl]: https://github.com/json4s/json4s#dsl-rules
[ec2]: https://aws.amazon.com/ec2/

[self-describing-jsons]: https://github.com/snowplow/iglu/wiki/Self-describing-JSONs
