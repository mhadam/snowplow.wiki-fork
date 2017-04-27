<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Trackers**](trackers) > Java Tracker

This page refers to version 0.4.0 of the Snowplow Java Tracker.

## Contents

- 1. [Overview](#overview)
- 2. [Initialization](#init)
  - 2.1 [Importing the module](#importing)
  - 2.2 [Creating a tracker](#create-tracker)
- 3. [Adding extra data](#add-data)
- 4. [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Custom contexts](#custom-contexts)
    - 4.1.2 [Optional timestamp argument](#tstamp-arg)
    - 4.1.3 [Tracker method return values](#return-values)
  - 4.2 [`trackScreenView()`](#screen-view)
  - 4.3 [`trackPageView()`](#page-view)
  - 4.4 [`trackEcommerceTransaction()`](#ecommerce-transaction)
    - 4.4.1 [`TransactionItem`](#ecommerce-transactionitem)
  - 4.5 [`trackStructuredEvent()`](#struct-event)
  - 4.6 [`trackUnstructuredEvent()`](#unstruct-event)
- 5 [Sending event](#emitter)
  - 5.1 [Using a buffer](#buffer)
  - 5.2 [Choosing the HTTP method](#http-method)
  - 5.3 [Method of sending HTTP requests](#http-request)
- 6 [Payload](#payload)
- 7 [Logging](#logging)

<a name="overview" />

## 1. Overview

The [Snowplow Java Tracker](https://github.com/snowplow/snowplow-java-tracker) allows you to track Snowplow events from your Java-based desktop and server apps, servlets and games. It supports JDK6+.

The tracker should be straightforward to use if you are comfortable with Java development; its API is modelled after Snowplow's [[Python Tracker]] so any prior experience with that tracker is helpful but not necessary. If you haven't already, have a look at the [[Java Tracker Setup]] guide before continuing.

[Back to top](#top)

<a name="init" />

## 2 Initialization

Assuming you have completed the [[Java Tracker Setup]] for your Java project, you are now ready to initialize the Java Tracker.

<a name="importing" />

### 2.1 Importing the module

Import the Java Tracker's classes into your Java code like so:

```java
import com.snowplowanalytics.snowplow.tracker.*;
```

That's it - you are now ready to initialize a Tracker instance.

[Back to top](#top)

<a name="create-tracker" />

### 2.2 Creating a Tracker

To instantiate a tracker in your code (can be global or local to the process being tracked) simply instantiate the `Tracker` interface with one of the following:

```java
Tracker(Emitter emitter, String namespace, String appId)
Tracker(Emitter emitter, Subject subject, String namespace, String appId)
Tracker(Emitter emitter, String namespace, String appId, boolean base64Encoded)
Tracker(Emitter emitter, Subject subject, String namespace, String appId, boolean base64Encoded)
```

For example:

```java
Tracker t1 = new Tracker("d3rkrsqld9gmqf.cloudfront.net", user1Subject, "AF003", "cloudfront", true);
Tracker t2 = new Tracker("d3rkrsqld9gmqf.cloudfront.net", "AF003", "cloudfront");
```

| **Argument Name** | **Description**                              |    **Required?**  |
|------------------:|:---------------------------------------------|:------------------|
| `emitter`         | The Emitter object you create                | Yes               |
| `subject`         | The subject that defines a user              | No                |
| `namespace`       | The name of the tracker instance             | Yes               |
| `appId`           | The application ID                           | Yes               |
| `base64Encoded`   | Whether to enable [base 64 encoding][base64] | No (Default true) |

[Back to top](#top)

<a name="add-data" />

## 3. Adding Subject data

You may have additional information about your application's environment, current user and so on, which you want to send to Snowplow with each event.

The Subject class has a set of `set...()` methods to attach extra data relating to the user to all tracked events:

* [`setPlatform`](#set-platform)
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
s1.setPlatform("cnsl");
s1.setScreenResolution(1920, 1080);
```

After that, you can add your Subject to your Tracker like so:
```java
Tracker(emitter, s1, namespace, appId);
// OR
t1.setSubject(s1);
```

<a name="set-platform" />

#### 3.1 Change the tracker's platform with `setPlatform`

You can change the platform the subject is using by calling:

```java
s1.setPlatform("cnsl");
```

For a full list of supported platforms, please see the [[Snowplow Tracker Protocol]].

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

If your Java code has access to the device's screen resolution, then you can pass this in to Snowplow too:

```java
t1.setScreenResolution( {{WIDTH}}, {{HEIGHT}} )
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```java
t1.setScreenResolution(1366, 768)
```

[Back to top](#top)

<a name="set-viewport-dimensions" />

### 3.4 Set viewport dimensions with `setViewport`

If your Java code has access to the viewport dimensions, then you can pass this in to Snowplow too:

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

If your Java code has access to the bit depth of the device's color palette for displaying images, then you can pass this in to Snowplow too:

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

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your websites and apps. We are constantly growing the range of functions available in order to capture that data more richly.

Tracking methods supported by the Java Tracker at a glance:

| **Function**                                                | **Description*                                         |
|------------------------------------------------------------:|:-------------------------------------------------------|
| [`trackScreenView()`](#screen-view)                         | Track the user viewing a screen within the application |
| [`trackPageView()`](#page-view)                             | Track and record views of web pages.                   |
| [`trackEcommerceTransaction()`](#ecommerce-transaction)     | Track an ecommerce transaction and its items           |
| [`trackStructuredEvent()`](#struct-event)                   | Track a Snowplow custom structured event               |
| [`trackUnstructuredEvent()`](#unstruct-event)               | Track a Snowplow custom unstructured event             |

<a name="common" />

### 4.1 Common

All events are tracked with specific methods on the tracker instance, of the form `trackXXX()`, where `XXX` is the name of the event to track.

<a name="custom-contexts" />

### 4.1.1 Custom contexts

In short, custom contexts let you add additional information about the circumstances surrounding an event in the form of a Java String in JSON format. dictionary object. Each tracking method accepts an additional optional contexts parameter after all the parameters specific to that method:

```java
t1.trackPageView(String pageUrl, String pageTitle, String referrer)
t1.trackPageView(String pageUrl, String pageTitle, String referrer, Map context)
t1.trackPageView(String pageUrl, String pageTitle, String referrer, double timestamp)
t1.trackPageView(String pageUrl, String pageTitle, String referrer, Map context, double timestamp)
```

The `context` argument should consist of a `Map` containing a JSON array of one or more contexts. The format of each individual context element is the same as for an [unstructured event](#unstruct-event).

If a visitor arrives on a page advertising a movie, the context dictionary might look like this:

```json
{
  "schema": "iglu:com.acme_company/movie_poster/jsonschema/2.1.1",
  "data": {
    "movie_name": "Solaris",
    "poster_country": "JP",
    "poster_year": "1978"
  }
}
```

Note that even if there is only one custom context attached to the event, it still needs to be placed in an array.

<a name="tstamp-arg" />

### 4.1.2 Optional timestamp & context argument

In all the trackers, we offer a way to set the timestamp if you want the event to show as tracked at a specific time. If you don't, we create a timestamp while the event is being tracked.

Here is an example:
```java

```

<a name="return-value" />

### 4.1.3 Tracker method return values

To be confirmed. As of now, trackers do not return anything.

<a name="screen-view" />

### 4.2 Track screen views with `trackScreenView()`

Use `trackScreenView()` to track a user viewing a screen (or equivalent) within your app. Arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `name`       | Human-readable name for this screen | Yes           | String                  |
| `id`         | Unique identifier for this screen   | No            | String                  |
| `context`    | Custom context for the event        | No            | String                  |
| `timestamp`  | Optional timestamp for the event    | No            | Long                    |

Example:

```java
t1.trackScreenView("HUD > Save Game", "screen23");
t1.trackScreenView("HUD > Save Game", contextMap, 123456);
```

[Back to top](#top)

<a name="page-view" />

### 4.3 Track pageviews with `trackPageView()`

If you are using Java servlet technology or similar to serve webpages to a browser, you can use `trackPageView()` to track a user viewing a page within your app.

Arguments are:

| **Argument** | **Description**                      | **Required?** | **Validation**          |
|-------------:|:-------------------------------------|:--------------|:------------------------|
| `page_url`   | The URL of the page                  | Yes           | String                  |
| `page_title` | The title of the page                | Yes           | String                  |
| `referrer`   | The address which linked to the page | Yes           | String                  |
| `context`    | Custom context for the event         | No            | Map                     |
| `timestamp`  | Optional timestamp for the event     | No            | Long                    |

Example:

```java
t1.trackPageView("www.example.com", "example", "www.referrer.com", mapContext);
t1.trackPageView("www.example.com", "example", "www.referrer.com");
```

[Back to top](#top)

<a name="ecommerce-transaction" />

### 4.4 Track ecommerce transactions with `trackEcommerceTransaction()`

Use `trackEcommerceTransaction()` to track an ecommerce transaction.

Arguments:

| **Argument**  | **Description**                      | **Required?** | **Validation**           |
|--------------:|:-------------------------------------|:--------------|:-------------------------|
| `order_id`    | ID of the eCommerce transaction      | Yes           | String                   |
| `total_value` | Total transaction value              | Yes           | Double                   |
| `affiliation` | Transaction affiliation              | Yes           | String                   |
| `tax_value`   | Transaction tax value                | Yes           | Double                   |
| `shipping`    | Delivery cost charged                | Yes           | Double                   |
| `city`        | Delivery address city                | Yes           | String                   |
| `state`       | Delivery address state               | Yes           | String                   |
| `country`     | Delivery address country             | Yes           | String                   |
| `currency`    | Transaction currency                 | Yes           | String                   |
| `items`       | Items in the transaction             | Yes           | List<TransactionItem>    |
| `context`     | Custom context for the event         | No            | Map                      |
| `timestamp`   | Optional timestamp for the event     | No            | Long                     |

The `items` argument is a `List` of individual `TransactionItem` elements representing the items in the e-commerce transaction. Note that `trackEcommerceTransaction` fires multiple events: one transaction event for the transaction as a whole, and one transaction item event for each element of the `items` `List`. Each transaction item event will have the same timestamp, order_id, and currency as the main transaction event.

[Back to top](#top)

<a name="ecommerce-transactionitem" />

### 4.4.1 Ecommerce TransactionItem with `trackEcommerceTransaction()`

To instantiate a TransactionItem in your code, simply use the following constructor signature:

```java
trackEcommerceTransactionItem(String order_id, String sku, Double price, Integer quantity, String name, String category, String currency, Map context, long transaction_id)
```

These are the fields that can appear as elements in each `TransactionItem` element of the transaction item `List`:

| **Field**  | **Description**                     | **Required?** | **Validation**           |
|-----------:|:------------------------------------|:--------------|:-------------------------|
| `order_id` | Order ID                            | Yes           | String                   |
| `sku`      | Item SKU                            | No            | String                   |
| `price`    | Item price                          | No            | double                   |
| `quantity` | Item quantity                       | No            | int                      |
| `name`     | Item name                           | No            | String                   |
| `category` | Item category                       | No            | String                   |
| `currency` | Item currency                       | No            | String                   |
| `context`  | Item context                        | No            | Map                      |
| `timestamp`| Optional timestamp for the event    | No            | Long                     |

Example of tracking a transaction containing two items:

```java
// Example to come, in the meantime here is the type signature:
t1.trackEcommerceTransaction(String order_id, Double total_value, String affiliation, Double tax_value,Double shipping, String city, String state, String country, String currency, List<TransactionItem> items, String context);
```

[Back to top](#top)

<a name="struct-event" />

### 4.5 Track structured events with `trackStructuredEvent()`

Use `trackStructuredEvent()` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument** | **Description**                                                  | **Required?** | **Validation** |
|-------------:|:---------------------------------------------------------------  |:--------------|:---------------|
| `category`   | The grouping of structured events which this `action` belongs to | Yes           | String         |
| `action`     | Defines the type of user interaction which this event involves   | Yes           | String         |
| `label`      | A string to provide additional dimensions to the event data      | Yes           | String         |
| `property`   | A string describing the object or the action performed on it     | Yes           | String         |
| `value`      | A value to provide numerical data about the event                | Yes           | Int            |
| `context`    | Custom context for the event                                     | No            | Map            |
| `timestamp`  | Optional timestamp for the event                                 | No            | Long           |

Example:

```java
t1.trackStructuredEvent("shop", "add-to-basket", "Add To Basket", "pcs", 2);
t1.trackStructuredEvent("shop", "add-to-basket", "Add To Basket", "pcs", 2, 123456.7);
```

[Back to top](#top)

<a name="unstruct-event" />

### 4.6 Track unstructured events with `trackUnstructuredEvent()`

trackUnstructuredEvent(String eventVendor, String eventName, Map<String, Object> dictInfo, String context)

Use `trackUnstructuredRvent()` to track a custom event which consists of a name and an unstructured set of properties. This is useful when:

* You want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow), or
* You want to track events which have unpredictable or frequently changing properties

The arguments are as follows:

| **Argument**   | **Description**                   |  **Required?** |    **Validation**   |
|---------------:|:----------------------------------|:---------------|:--------------------|
| `eventData`    | The properties of the event       | Yes            | Map<String, Object> |
| `context`      | Custom context for the event      | No             | Map                 |
| `timestamp`    | Optional timestamp for the event  | No             | Long                |

The `eventData` must be either a `String` or a `Map<String, Object>`.

If you supply a `String`, make sure that it is a valid JSON object containing two fields: `schema` and `data`. `data` is itself a JSON object containing the properties of the unstructured event. `schema` identifies the JSON schema against which `data` should be validated.

Example:

```java
// Example to come, in the meantime here is the type signature:
t1.trackUnstructuredEvent(String eventVendor, String eventName, String eventData, String context);
```

If you supply a `Map<String, Object>`, make sure that this top-level contains your `schema` and `data` keys, and then store your `data` properties as a child `Map<String, Object>`.

Example:

```java
// Example to come, in the meantime here is the type signature:
t1.trackUnstructuredEvent(String eventVendor, String eventName, Map<String, Object> eventData, String context);
```

For more on JSON schema, see the [blog post][self-describing-jsons].

<a name="emitter" />

## 5. Sending event

Events are sent using an `Emitter` class. You can initialize an class with a collector endpoint URL with various options to choose how these events should be sent.
Here are the Emitter interfaces that can be used:

```java
Emitter(String URI)
Emitter(String URI, HttpMethod httpMethod)
```

For example:

```java
Emitter e1 = new Emitter("d3rkrsqld9gmqf.cloudfront.net");
Emitter e2 = new Emitter("d3rkrsqld9gmqf.cloudfront.net", HttpMethod.POST);
```

| **Argument Name** | **Description**                                   |    **Required?**  |
|------------------:|:--------------------------------------------------|:------------------|
| `URI`             | The collector endpoint URI events will be sent to | Yes               |
| `httpMethod`      | The HTTP method events should be sent             | No                |

<a name="buffer" />

### 5.1 Using a buffer

A buffer is used to group events together in bulk before sending them. This is especially handy to reduce network usage. By default, the Emitter buffers up to 10 events before sending them. You can change this to send evenets instantly as soon as they are created like so:

```java
Emitter e1 = new Emitter("d3rkrsqld9gmqf.cloudfront.net");
e1.setBufferOption(BufferOption.Instant);
// OR
e1.setBufferOption(BufferOption.Default);
```

Here are all the posibile options that you can use:
|  **Option**  | **Description**                                    |
|-------------:|:---------------------------------------------------|
| `Instant`    | Events are sent as soon as they are created        |
| `Default`    | Sends events in a group when 10 events are created |

<a name="http-method" />

###  5.2 Choosing the HTTP method

Snowplow supports receiving events via GET requests, but will soon have POST support. In a GET request, each event is sent in individual request. With POST requests, events are bundled together in one request.

You can set the HTTP method in the Emitter constructor:
```java
Emitter e1 = new Emitter("d3rkrsqld9gmqf.cloudfront.net", HttpMethod.POST);
```

Here are all the posibile options that you can use:
|  **Option**  | **Description**                                    |
|-------------:|:---------------------------------------------------|
| `GET`        | Sends events as GET requests                       |
| `POST`       | Sends events as POST requests                      |

<a name="http-request" />

###  5.3 Method of sending HTTP requests

An Emitter sends requests synchronously by default. If you want events to be sent asynchronously you can set this using `setRequestMethod(RequestMethod)`:

```java
Emitter e1 = new Emitter("d3rkrsqld9gmqf.cloudfront.net");
e1.setRequestMethod(RequestMethod.Asynchronous);
```

Here are all the posibile options that you can use:
|  **Option**    | **Description**               |
|---------------:|:------------------------------|
| `Synchronous`  | Sends events synchronously    |
| `Asynchronous` | Sends events asynchronously   |


<a name="payload" />

## 6. Payload

TBD

<a name="logging" />

## 7. Logging

Logging in the Tracker is done using SLF4J. Majority of the logging set as `DEBUG` so will not overly populate your own logging.

[documentation]: https://gleasonk.github.io/Saggezza/JavaDoc/index.html

[jsonschema]: http://snowplowanalytics.com/blog/2014/05/13/introducing-schemaver-for-semantic-versioning-of-schemas/
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
