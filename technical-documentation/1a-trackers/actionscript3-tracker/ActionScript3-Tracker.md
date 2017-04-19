<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Trackers**](trackers) » Actionscript 3 (AS3) Tracker

## Contents

- 1. [Overview](#overview)  
- 2. [Initialization](#init)  
  - 2.1 [Importing the module](#importing)
  - 2.2 [Creating a tracker](#create-tracker)
    - 2.2.1 [`emitter`](#emitter)  
    - 2.2.2 [`subject`](#subject)
    - 2.2.3 [`namespace`](#namespace)
    - 2.2.4 [`appId`](#app-id)
    - 2.2.5 [`base64Encoded`](#base64)
    - 2.2.6 [`setPlatform`](#set-platform)
- 3. [Adding extra data: the Subject class](#add-data)
  - 3.2 [`setUserId`](#set-user-id)
  - 3.3 [`setScreenResolution`](#set-screen-resolution)
  - 3.4 [`setViewport`](#set-viewport-dimensions)
  - 3.5 [`setColorDepth`](#set-color-depth)
  - 3.6 [`setTimezone`](#set-timezone)
  - 3.7 [`setLanguage`](#set-lang)
- 4. [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Custom contexts](#custom-contexts)
    - 4.1.2 [Optional timestamp argument](#tstamp-arg)
    - 4.1.3 [Tracker method return values](#return-values)
  - 4.2 [`trackPageView()`](#page-view)
  - 4.3 [`trackScreenView()`](#screen-view)
  - 4.4 [`trackEcommerceTransaction()`](#ecommerce-transaction)
    - 4.4.1 [`TransactionItem`](#ecommerce-transactionitem)
  - 4.5 [`trackStructuredEvent()`](#struct-event)
  - 4.6 [`trackUnstructuredEvent()`](#unstruct-event)
- 5 [Sending event: `Emitter`](#emitters)
  - 5.1 [Using a buffer](#buffer)
  - 5.2 [Choosing the HTTP method](#http-method)
  - 5.3 [Method of sending HTTP requests](#http-request)
  - 5.4 [Emitter callback](#http-callback)
- 6 [Payload](#payload)
  - 6.1 [Tracker Payload](#tracker-payload)
  - 6.2 [Schema Payload](#schema-payload)

<a name="overview" />

## 1. Overview

The [Snowplow Actionscript 3 (AS3) Tracker](https://github.com/snowplow/snowplow-actionscript3-tracker) allows you to track Snowplow events from a Flash movie, Flex application or Adobe AIR application.

The tracker should be straightforward to use if you are comfortable with AS3 development.

[Back to top](#top)

<a name="init" />

## 2 Initialization

Assuming you have completed the [Actionscript 3 Tracker Setup](./ActionScript3-Tracker-Setup) for your project, you are now ready to initialize the AS3 Tracker.

<a name="importing" />

### 2.1 Importing the module

Import the AS3 Tracker's classes into your AS3 code like so:

```java
import com.snowplowanalytics.snowplow.tracker.*;
import com.snowplowanalytics.snowplow.tracker.emitter.*;
import com.snowplowanalytics.snowplow.tracker.payload.*;
```

That's it - you are now ready to initialize a Tracker instance. 

[Back to top](#top)

<a name="create-tracker" />

### 2.2 Creating a Tracker

To instantiate a tracker in your code simply instantiate the `Tracker` interface:

```java
Tracker(emitter:Emitter, namespace:String, appId:String, subject:Subject, stage:Stage = null, base64Encoded:Boolean = true)
```

For example:

```java
var t1:Tracker = new Tracker(emitter, "AF003", "cf", user1Subject, this.stage, true);
```

| **Argument Name** | **Description**                              |    **Required?**  |
|------------------:|:---------------------------------------------|:------------------|
| `emitter`         | The Emitter object you create                | Yes               |
| `namespace`       | The name of the Tracker instance             | Yes               |
| `appId`           | The application ID                           | Yes               |
| `subject`         | The Subject that defines a user              | No (default null) |
| `stage`           | The stage property of a DisplayObject        | No (Default null) |
| `base64Encoded`   | Whether to enable [base 64 encoding][base64] | No (Default true) |

<a name="emitter" />

#### 2.2.1 `emitter`

The emitter to which the tracker will send events. See [Emitters](#5-sending-event-emitter) for more on emitter configuration.

<a name="subject" />

#### 2.2.2 `subject`

The user which the Tracker will track. This should be an instance of the [Subject](#3-adding-extra-data-the-subject-class) class. You don't need to set this during Tracker construction; you can use the `Tracker.setSubject` method afterwards. In fact, you don't need to create a subject at all. If you don't, though, your events won't contain user-specific data such as timezone and language.


<a name="namespace" />

#### 2.2.3 `namespace`

If provided, the `namespace` argument will be attached to every event fired by the new tracker. This allows you to later identify which tracker fired which event if you have multiple trackers running.

<a name="app-id" />

#### 2.2.4 `appId`

The `appId` argument lets you set the application ID to any string.

<a name="base64" />

#### 2.2.5 `base64Encoded`

By default, unstructured events and custom contexts are encoded into Base64 to ensure that no data is lost or corrupted. You can turn encoding on or off using the Boolean `base64Encoded` argument.

[Back to top](#top)

<a name="set-platform" />

### 2.2.6 Change the tracker's platform with `setPlatform`

You can change the platform by calling:

```java
tracker.setPlatform("cnsl");
```

For a full list of supported platforms, please see the [[Snowplow Tracker Protocol]].

[Back to top](#top)

<a name="add-data" />

## 3. Adding extra data: the Subject class

You may have additional information about your application's environment, current user and so on, which you want to send to Snowplow with each event.

The Subject class has a set of `set...()` methods to attach extra data relating to the user to all tracked events:

* [`setUserId`](#set-user-id)
* [`setScreenResolution`](#set-screen-resolution)
* [`setViewport`](#set-viewport-dimensions)
* [`setColorDepth`](#set-color-depth)
* [`setTimezone`](#set-timezone)
* [`setLanguage`](#set-lang)

Here are some examples:

```java
s1.setUserID("Kevin Gleason"); 
s1.setLanguage("en-gb");
s1.setScreenResolution(1920, 1080);
```

After that, you can add your Subject to your Tracker like so:
```java
Tracker(emitter, namespace, appId, s1);
// OR
t1.setSubject(s1);
```

[Back to top](#top)

<a name="set-user-id" />

### 3.2 Set user ID with `setUserId`

You can set the user ID to any string:

```java
s1.setUserId( "{{USER ID}}" )
```

Example:

```java
s1.setUserId("alexd")
```

[Back to top](#top)

<a name="set-screen-resolution" />

### 3.3 Set screen resolution with `setScreenResolution`

If you wish to track the device's screen resolution, then you can pass this in to Snowplow too:

```java
t1.setScreenResolution( {{WIDTH}}, {{HEIGHT}} )
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```java
t1.setScreenResolution(1366, 768)
```

You can get these values by referencing [flash.system.Capabilities.screenResolutionX][screenResolutionX] and [flash.system.Capabilities.screenResolutionY][screenResolutionY], although they will only reflect the dimensions of the main screen.

Alternatively, if your AS3 code has script access via [ExternalInterface][externalInterface], you can use Javascript to query these values from the browser or another player context.

[Back to top](#top)

<a name="set-viewport-dimensions" />

### 3.4 Set viewport dimensions with `setViewport`

If your AS3 code has access to the viewport dimensions (by calling Javascript code through [ExternalInterface][externalInterface]), then you can pass this in to Snowplow too:

```java
s.setViewport( {{WIDTH}}, {{HEIGHT}} )
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```java
s.setViewport(300, 200)
```

[Back to top](#top)

<a name="set-color-depth" />

### 3.5 Set color depth with `setColorDepth`

If your AS3 code has access to the bit depth of the device's color palette for displaying images (by calling Javascript's screen.colorDepth via [ExternalInterface][externalInterface]), then you can pass this in to Snowplow too:

```java
s.setColorDepth( {{BITS PER PIXEL}} )
```

The number should be a positive integer, in bits per pixel. Example:

```java
s.setColorDepth(32)
```

[Back to top](#top)

<a name="set-timezone" />

### 3.6 Set timezone with `setTimezone`

This method lets you pass a user's timezone in to Snowplow:

```java
s.setTimezone( {{TIMEZONE}} )
```

The timezone should be a string:

```java
s.setTimezone("Europe/London")
```

[Back to top](#top)

<a name="set-lang" />

### 3.7 Set the language with `setLanguage`

This method lets you pass a user's language in to Snowplow:

```java
s.setLanguage( {{LANGUAGE}} )
```

The language should be a string:

```java
s.setLanguage('en')
```

[Back to top](#top)


<a name="events" />

## 4. Tracking specific events

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your Flash apps. We are constantly growing the range of functions available in order to capture that data more richly.

Tracking methods supported by the AS3 Tracker at a glance:

| **Function**                                                | **Description*                                         |
|------------------------------------------------------------:|:-------------------------------------------------------|
| [`trackScreenView()`](#screen-view)                         | Track the user viewing a screen within the application |
| [`trackEcommerceTransaction()`](#ecommerce-transaction)     | Track an ecommerce transaction and its items           |
| [`trackStructuredEvent()`](#struct-event)                   | Track a Snowplow custom structured event               |
| [`trackUnstructuredEvent()`](#unstruct-event)               | Track a Snowplow custom unstructured event             |

[Back to top](#top)

<a name="common" />

### 4.1 Common

All events are tracked with specific methods on the tracker instance, of the form `trackXXX()`, where `XXX` is the name of the event to track.

[Back to top](#top)

<a name="custom-contexts" />

### 4.1.1 Custom contexts

In short, custom contexts let you add additional information about the circumstances surrounding an event in the form of an Object. Each tracking method accepts an additional optional contexts parameter after all the parameters specific to that method:

```java
t1.trackScreenView(name:String, id:String, context:Array, timestamp:Number);
```

The `context` argument should consist of a `Array` of `SchemaPayload` representing an array of one or more contexts. The format of each individual context element is the same as for an [unstructured event](#unstruct-event).

If a visitor arrives on a page advertising a movie, the context dictionary might look like this:

```json
{ 
  "schema": "iglu:com.acme_company/movie_poster/jsonschema/2-1-1",
  "data": {
    "movie_name": "Solaris", 
    "poster_country": "JP", 
    "poster_year": "1978"
  }
}
```

Note that even if there is only one custom context attached to the event, it still needs to be placed in an Array.

[Back to top](#top)

<a name="tstamp-arg" />

### 4.1.2 Optional timestamp & context argument

In all the trackers, we offer a way to set the timestamp if you want the event to show as tracked at a specific time. If you don't, we create a timestamp while the event is being tracked.

Here is an example:
```java
t1.trackScreenView("HUD > Save Game", "screen23", contextList, 123456789);
```

[Back to top](#top)

<a name="return-value" />

### 4.1.3 Tracker method return values

To be confirmed. As of now, trackers do not return anything.

[Back to top](#top)

<a name="page-view" />

### 4.2 Track page views with `trackPageView()`

Use `trackPageView()` to track a user viewing a screen within your Flash app, in the case where the screen behaves like a web page and has its own unique URL that appears in the browser's address bar. Arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `name`       | Human-readable name for this screen | No            | String                  |
| `id`         | Unique identifier for this screen   | No            | String                  |
| `context`    | Custom context for the event        | No            | Array<SchemaPayload>    |
| `timestamp`  | Optional timestamp for the event    | No            | Number                  |

Example:

```java
tracker.trackPageView("www.mysite.com#page3", "Page Three", "www.me.com", contextList);
```

In Flash, the more common case is to use [trackScreenView](#screen-view).

[Back to top](#top)

<a name="screen-view" />

### 4.3 Track screen views with `trackScreenView()`

Use `trackScreenView()` to track a user viewing a screen (or equivalent) within your app. Arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `name`       | Human-readable name for this screen | No            | String                  |
| `id`         | Unique identifier for this screen   | No            | String                  |
| `context`    | Custom context for the event        | No            | Array<SchemaPayload>    |
| `timestamp`  | Optional timestamp for the event    | No            | Number                  |

Example:

```java
t1.trackScreenView("HUD > Save Game", "screen23", contextList, 123456);
```

[Back to top](#top)

<a name="ecommerce-transaction" />

### 4.4 Track ecommerce transactions with `trackEcommerceTransaction()`

Use `trackEcommerceTransaction()` to track an ecommerce transaction.

Arguments:

| **Argument**  | **Description**                      | **Required?** | **Validation**           |
|--------------:|:-------------------------------------|:--------------|:-------------------------|
| `order_id`    | ID of the eCommerce transaction      | Yes           | String                   |
| `total_value` | Total transaction value              | Yes           | Number                   |
| `affiliation` | Transaction affiliation              | Yes           | String                   |
| `tax_value`   | Transaction tax value                | Yes           | Number                   |
| `shipping`    | Delivery cost charged                | Yes           | Number                   |
| `city`        | Delivery address city                | Yes           | String                   |
| `state`       | Delivery address state               | Yes           | String                   |
| `country`     | Delivery address country             | Yes           | String                   | 
| `currency`    | Transaction currency                 | Yes           | String                   |
| `items`       | Items in the transaction             | Yes           | Array<TransactionItem>   |
| `context`     | Custom context for the event         | No            | Object                   |
| `timestamp`   | Optional timestamp for the event     | No            | Number                   |

The `items` argument is a `List` of individual `TransactionItem` elements representing the items in the e-commerce transaction. Note that `trackEcommerceTransaction` fires multiple events: one transaction event for the transaction as a whole, and one transaction item event for each element of the `items` `List`. Each transaction item event will have the same timestamp, order_id, and currency as the main transaction event.

[Back to top](#top)

<a name="ecommerce-transactionitem" />

### 4.4.1 Ecommerce TransactionItem with `trackEcommerceTransaction()`

To instantiate a TransactionItem in your code, simply use the following constructor signature:

```java
trackEcommerceTransactionItem(order_id:String, sku:String, price:Number, quantity:int, name:String, category:String, currency:String, context:Array, timestamp:Number)
```

These are the fields that can appear as elements in each `TransactionItem` element of the transaction item `Array`:

| **Field**  | **Description**                     | **Required?** | **Validation**           |
|-----------:|:------------------------------------|:--------------|:-------------------------|
| `order_id` | Order ID                            | Yes           | String                   |
| `sku`      | Item SKU                            | No            | String                   |
| `price`    | Item price                          | No            | Number                   |
| `quantity` | Item quantity                       | No            | int                      |
| `name`     | Item name                           | No            | String                   |
| `category` | Item category                       | No            | String                   |
| `currency` | Item currency                       | No            | String                   |
| `context`  | Item context                        | No            | Array<SchemaPayload>       |
| `timestamp`| Optional timestamp for the event    | No            | Number                     |

Example of tracking a transaction containing two items:

```java
// Example to come, in the meantime here is the type signature:
t1.trackEcommerceTransaction(order_id:String, total_value:Number, affiliation:String, tax_value:Number, shipping:Number, city:String, state:String, country:String, currency:String, items:Array, context:Array = null, timestamp:Number = 0);
t1.trackEcommerceTransaction("6a8078be", 300, "my_affiliate", 30, 10, "Boston", "Massachusetts", "USA", "USD", items, context);
```

[Back to top](#top)

<a name="struct-event" />

### 4.5 Track structured events with `trackStructuredEvent()`

Use `trackStructuredEvent()` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument** | **Description**                                                  | **Required?** | **Validation**    |
|-------------:|:---------------------------------------------------------------  |:--------------|:------------------|
| `category`   | The grouping of structured events which this `action` belongs to | Yes           | String            |
| `action`     | Defines the type of user interaction which this event involves   | Yes           | String            |
| `label`      | A string to provide additional dimensions to the event data      | Yes           | String            |
| `property`   | A string describing the object or the action performed on it     | Yes           | String            |
| `value`      | A value to provide numerical data about the event                | Yes           | int               |
| `context`    | Custom context for the event                                     | No            | Array<SchemaPayload>|
| `timestamp`  | Optional timestamp for the event                                 | No            | Number              |

Example:

```java
t1.trackStructuredEvent(category:String, action:String, label:String, property:String, value:int, context:Array = null, timestamp:Number = 0);
```

[Back to top](#top)

<a name="unstruct-event" />

### 4.6 Track unstructured events with `trackUnstructuredEvent()`

Custom unstructured events are a flexible tool that enables Snowplow users to define their own event types and send them into Snowplow.

When a user sends in a custom unstructured event, they do so as a JSON of name-value properties, that conforms to a JSON schema defined for the event earlier.

Use `trackUnstructuredEvent()` to track a custom event which consists of a name and an unstructured set of properties. This is useful when:

* You want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow), or
* You want to track events which have unpredictable or frequently changing properties

The arguments are as follows:

| **Argument**   | **Description**                   |  **Required?** |    **Validation**   |
|---------------:|:----------------------------------|:---------------|:--------------------|
| `eventData`    | The properties of the event       | Yes            | SchemaPayload       |
| `context`      | Custom context for the event      | No             | Array<SchemaPayload>|
| `timestamp`    | Optional timestamp for the event  | No             | Number              |

Example:

```java
t1.trackUnstructuredEvent(eventData, contextList);
```

For more on JSON schema, see the [blog post] [self-describing-jsons].

[Back to top](#top)

<a name="emitters" />

## 5. Sending event: `Emitter`

Events are sent using an `Emitter` class. You can initialize a class with a collector endpoint URL and optionally choose the HTTP `POST` method instead of `GET`.

```java
Emitter(uri:string, httpMethod:String)
```

For example:

```java
var e1:Emitter = new Emitter("d3rkrsqld9gmqf.cloudfront.net");
var e2:Emitter = new Emitter("d3rkrsqld9gmqf.cloudfront.net", URLRequestMethod.POST);
```

| **Argument Name** | **Description**                                                             |    **Required?**  |
|------------------:|:----------------------------------------------------------------------------|:------------------|
| `uri`             | The collector endpoint URI events will be sent to                           | Yes               |
| `httpMethod`      | The HTTP method to use when sending events                                  | No                |


<a name="buffer" />
### 5.1 Using a buffer

A buffer is used to group events together in bulk before sending them. This is useful to reduce network usage. By default, the AS3 Emitter does not buffer but instead sends events instantly as soon as they are created. You can use a buffer instead by calling setBufferOption with the number of events to batch together in the buffer:

```java
var e1:Emitter = new Emitter("d3rkrsqld9gmqf.cloudfront.net");
e1.setBufferOption(BufferOption.BATCH);
```

There are two predefined constants but you can use any integer value:

|  **Option**             | **Description**                                        |
|------------------------:|:-------------------------------------------------------|
| BufferOption.DEFAULT    | 1. Events are sent as soon as they are created         |
| BufferOption.BATCH      | 10. Sends events in a group when 10 events are created |

[Back to top](#top)

<a name="http-method" />

###  5.2 Choosing the HTTP method

Snowplow supports receiving events via `GET` and `POST` requests. In a `GET` request, each event is sent in an individual request. With POST requests, events are bundled together in one request.

You can set the HTTP method in the Emitter constructor:
```java
var e1:Emitter = new Emitter("d3rkrsqld9gmqf.cloudfront.net", URLRequestMethod.POST);
```

The default method is `GET`.

Actionscript provides standard constants for the two options in the URLRequestMethod class:

URLRequestMethod.GET
URLRequestMethod.POST

[Back to top](#top)

<a name="http-request" />

###  5.3 Method of sending HTTP requests

An AS3 Emitter sends requests asynchronously. Flash does not support synchronous requests.

[Back to top](#top)

<a name="http-callback" />

###  5.4 Emitter callback

The AS3 Emitter dispatches events when it succeeds or fails to flush the buffer. 

You can register event listeners for these events if you need to handle the success or failure case. Here is a sample bit of code to show how it could work:

```java
var emitter:Emitter = new Emitter(testURL);
emitter.addEventListener(EmitterEvent.SUCCESS, function(bufferLength:int) {
    trace("Buffer length for POST/GET:" + bufferLength);
  }
);
emitter.addEventListener(EmitterEvent.FAILURE, function (successCount:int, failedEvents:Array, errorString:String) {
    trace("Failure, successCount: " + successCount + "\nfailedEvents:\n" + failedEvents.toString() + "\nerror:\n" + errorString:String);
  }
);
```

In the example, we can see in-line handling of the both cases. If events are all successfully sent, the success callback method receives the number of successful events sent. If there were any failures, the failure callback method receives the number of successful events sent (if any) and a *Array of events* that failed to be sent (i.e. the HTTP state code did not return 200). In addition, in the case of failure, an additional parameter is provided with the text of the network or security error that caused the failure.

[Back to top](#top)

<a name="payload" />

## 6. Payload

A Payload interface is used for implementing a [TrackerPayload](#tracker-payload) and [SchemaPayload](#schema-payload), but accordingly, can be used to implement your own Payload class if you choose.

[Back to top](#top)

<a name="tracker-payload" />

### 6.1 Tracker Payload

A TrackerPayload is used internally within the AS3 Tracker to create the tracking event payloads that are passed to an Emitter to be sent accordingly.

[Back to top](#top)

<a name="schema-payload" />

### 6.2 Schema Payload

A SchemaPayload is used primarily as a wrapper around a TrackerPayload. After creating a TrackerPayload, you create a SchemaPayload and use `setData` with the Payload, followed by, `setSchema` to set the schema that the payload will be used against.

This is mainly used under the hood, in the Tracker class but is useful to know if you want to create your own Tracker class.

Here's a short example:
```java
// This is our TrackerPayload that we created
var payload:IPayload = new TrackerPayload();
trackerPayload.add("key", "value");

// We wrap that payload in a SchemaPayload before sending it.
var payload:SchemaPayload = new SchemaPayload();
payload.setSchema(Constants.SCHEMA_SCREEN_VIEW);
payload.setData(trackerPayload);
```

[Back to top](#top)

[screenResolutionX]: http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/system/Capabilities.html#screenResolutionX
[screenResolutionY]: http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/system/Capabilities.html#screenResolutionY
[externalInterface]: http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/external/ExternalInterface.html
[base64]: https://en.wikipedia.org/wiki/Base64
