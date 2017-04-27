<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Trackers**](trackers) > Node.js tracker

*This page refers to version 0.2.0 of the Snowplow Node.js Tracker. Click [here][v1] for the corresponding documentation for version 0.1.0.*

## Contents

- 1. [Overview](#overview)
- 2. [Initialization](#init)
- 3. [Configuration](#config)
  - 3.1 [`setPlatform()`](#set-platform)
  - 3.2 [`setUserId()`](#set-user-id)
  - 3.3 [`setScreenResolution()`](#set-screen-resolution)
  - 3.4 [`setViewport()`](#set-viewport)
  - 3.5 [`setColorDepth()`](#set-color-depth)
  - 3.6 [`setTimezone()`](#set-timezone)
  - 3.7 [`setLang()`](#set-lang)
- 4. [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Optional arguments](#optional-arguments)
    - 4.1.2 [Custom contexts](#custom-contexts)
    - 4.1.3 [Timestamps](#tstamp-arg)
  - 4.2 [`trackScreenView()`](#screen-view)
  - 4.3 [`trackPageView()`](#page-view)
  - 4.4 [`trackEcommerceTransaction()`](#ecommerce-transaction)
  - 4.5 [`trackStructEvent()`](#struct-event)
  - 4.6 [`trackUnstructEvent()`](#unstruct-event)

<a name="overview" />

## 1. Overview

The [Snowplow Node.js Tracker][repo] allows you to track Snowplow events from your Node.js apps.

The tracker should be straightforward to use if you are comfortable with JavaScript development; any prior experience with other Snowplow trackers, Google Analytics or Mixpanel (which have similar APIs to Snowplow) is helpful but not necessary.

<a name="init" />

## 2 Initialization

Assuming you have completed the [Node.js Tracker Setup](Node.js-tracker-setup) for your project, you are now ready to initialize the Tracker.

Require the Node.js Tracker module into your code like so:

```js
var snowplow = require('snowplow-tracker');
var emitter = snowplow.emitter;
var tracker = snowplow.tracker;
```

First, initialize an emitter instance. This object will be responsible for how and when events are sent to Snowplow.

```js
var e = emitter(
  'myscalastreamcollector.net', // Collector endpoint
  'http', // Optionally specify a method - http is the default
  8080, // Optionally specify a port
  'POST', // Method - defaults to GET
  5, // Only send events once n are buffered. Defaults to 1 for GET requests and 10 for POST requests.
  function (error, body, response) { // Callback called for each request
    if (error) {
      console.log("Request to Scala Stream Collector failed!");
    }
  }
);
```

Initialise a tracker instance like this:

```js
var t = tracker([e], 'myTracker', 'myApp', false);
```

The `tracker` function takes four parameters:

* An array of emitters to which the tracker will hand Snowplow events
* An optional tracker `namespace` which will be attached to all events which the tracker fires, allowing you to identify their origin
* The `appId`, or application ID
* `encodeBase64`, which determines whether unstructured events and custom contexts will be base 64 encoded (by default they are).

[Back to top](#top)

<a name="config" />

## 3. Configuration

You may have additional information about your application"s environment, current user and so on, which you want to send to Snowplow with each event.

The tracker instance has a set of `set...()` methods to attach extra data to all tracked events:

* [`setPlatform()`](#set-platform)
* [`setUserId()`](#set-user-id)
* [`setScreenResolution()`](#set-screen-resolution)
* [`setViewport`](#set-viewport)
* [`setColorDepth()`](#set-color-depth)
* [`setTimezone()`](#set-timezone)
* [`setLang()`](#set-lang)

We will discuss each of these in turn below:

<a name="set-platform" />

### 3.1 Set the platform ID with `setPlatform()`

You can set the platform:

```js
t.setPlatform( "{{PLATFORM}}" );
```

Example:

```js
t.setPlatform("mob");
```

If the platform is not set manually, it defaults to `'srv'` (for server).

For a full list of supported platforms, please see the [[Snowplow Tracker Protocol]].

[Back to top](#top)

<a name="set-user-id" />

### 3.2 Set user ID with `setUserId()`

You can set the user ID to any string:

```js
t.setUserId( "{{USER ID}}" );
```

Example:

```js
t.setUserId("alexd");
```

[Back to top](#top)

<a name="set-screen-resolution" />

### 3.3 Set screen resolution with `setScreenResolution()`

If your code has access to the device's screen resolution, then you can pass this in to Snowplow too:

```js
t.setScreenResolution( {{WIDTH}}, {{HEIGHT}} );
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```js
t.setScreenResolution(1366, 768);
```

[Back to top](#top)

<a name="set-viewport" />

### 3.4 Set viewport dimensions with `setViewport()`

If your code has access to the device's screen resolution, then you can pass this in to Snowplow too:

```js
t.setViewport( {{WIDTH}}, {{HEIGHT}} );
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```js
t.setViewport(300, 200);
```

[Back to top](#top)

<a name="set-color-depth" />

### 3.5 Set color depth with `setColorDepth()`

If your code has access to the bit depth of the device's color palette for displaying images, then you can pass this in to Snowplow too:

```js
t.setColorDepth( {{BITS PER PIXEL}} );
```

The number should be a positive integer, in bits per pixel. Example:

```js
t.setColorDepth(32);
```

[Back to top](#top)

<a name="set-timezone" />

### 3.6 Set the timezone with `setTimezone()`

This method lets you pass a user's language in to Snowplow:

```js
t.setTimezone( {{TIMEZONE}} );
```

Example:

```js
t.setTimezone('Europe/London');
```

[Back to top](#top)

<a name="set-lang" />

### 3.7 Set the language with `setLang()`

This method lets you pass a user's language in to Snowplow:

```js
t.setLang( {{LANGUAGE}} );
```

The number should be a positive integer, in bits per pixel. Example:

```js
t.setLang('en');
```

[Back to top](#top)


<a name="events" />

## 4. Tracking specific events

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your websites and apps. We are constantly growing the range of functions available in order to capture that data more richly.

Tracking methods supported by the Node.js Tracker at a glance:

| **Function**                                  | **Description**                                        |
|----------------------------------------------:|:-------------------------------------------------------|
| [`trackScreenView()`](#screen-view)         | Track the user viewing a screen within the application |
| [`trackPageView()`](#page-view)             | Track and record views of web pages.                   |
| [`trackEcommerceTransaction()`](#ecommerce-transaction)   | Track an ecommerce transaction           |
| [`trackStructEvent()`](#struct-event)       | Track a Snowplow custom structured event               |
| [`trackUnstructEvent()`](#unstruct-event)   | Track a Snowplow custom unstructured event             |

Details of other tracking methods are available in the documentation for the [tracker core][tracker-core].

<a name="common" />

### 4.1 Common

All events are tracked with specific methods on the tracker instance, of the form `trackXXX()`, where `XXX` is the name of the event to track.

<a name="optional-arguments" />

### 4.1.1 Optional arguments

Each tracker method has both default and optional arguments. If you don't want to provide a value for an optional argument, just pass `null` and it will be ignored. For example, if you want to track a page view event with a referrer but without a title:

``js
t.trackPageView('http://www.example.com', null, 'http://www.referer.com');
``

<a name="custom-contexts" />

### 4.1.2 Custom contexts

In short, custom contexts let you add additional information about the circumstances surrounding an event in the form of a JavaScript dictionary object. Each tracking method accepts an additional optional contexts parameter after all the parameters specific to that method:

```js
t.trackPageView('myUrl', 'myPage', 'myReferrer', myCustomContexts);
```

The `context` argument should consist of an array of one or more dictionaries. Each dictionary should be a self-describing JSON following the same pattern as an [unstructured event](#unstruct-event).

**Important:** Even if only one custom context is being attached to an event, it still needs to be wrapped in an array.

If a visitor arrives on a page advertising a movie, the context dictionary might look like this:

```js
{
  "schema": "iglu:com.acme_company/movie_poster/jsonschema/2-1-1",
  "data": {
    "movie_name": "Solaris",
    "poster_country": "JP"
  }
}
```

This is how to fire a page view event with the above custom context:

```js
t.trackPageView("http://www.films.com", "Homepage", null, [{
  "schema": "iglu:com.acme_company/movie_poster/jsonschema/2-1-1",
  "data": {
    "movie_name": "Solaris",
    "poster_country": "JP"
  }
}]);
```

Note that even though there is only one custom context attached to the event, it still needs to be placed in an array.

<a name="tstamp-arg" />

### 4.1.3 Timestamps

Each `track...()` method supports an optional timestamp as its final argument, after the context argument; this allows you to manually override the timestamp attached to this event.

If you do not pass this timestamp in as an argument, then the Tracker will use the current time to be the timestamp for the event.

Here is an example tracking a structured event and supplying the optional timestamp argument. We can explicitly supply `null`s for the intervening arguments which are empty:

```js
t.trackStructEvent("game action", "save", null, null, null, 1368725287000);
```

Timestamp is counted in milliseconds since the Unix epoch - the same format as generated by `new Date().getTime()`.

[Back to top](#top)

<a name="screen-view" />

### 4.2 Track screen views with `trackScreenView()`

Use `trackScreenView()` to track a user viewing a screen (or equivalent) within your app. Arguments are:

| **Argument** | **Description**                     | **Required?** | **Type**                |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `name`       | Human-readable name for this screen | No            | Non-empty string        |
| `id`         | Unique identifier for this screen   | No            | String                  |
| `context`    | Custom context                      | No            | Array                    |
| `tstamp`     | When the screen was viewed          | No            | Positive integer        |

`name` and `id` are not individually required, but you must provide at least one of them.

Example:

```js
t.trackScreenView("HUD > Save Game", "screen23", null, 1368725287000);
```

[Back to top](#top)

<a name="page-view" />

### 4.3 Track pageviews with `trackPageView()`

Use `trackPageView()` to track a user viewing a page within your app.
Arguments are:

| **Argument** | **Description**                     | **Required?** | **Type**                |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `pageUrl`    | The URL of the page                 | Yes           | Non-empty string        |
| `pageTitle`  | The title of the page               | No            | String                  |
| `referrer`   | The address which linked to the page| No            | String                  |
| `context`    | Custom context                      | No            | Array                    |
| `tstamp`     | When the screen was viewed          | No            | Positive integer        |

Example:

```js
t.trackPageView("www.example.com", "example", "www.referrer.com");
```

[Back to top](#top)

<a name="ecommerce-transaction" />

### 4.4 Track ecommerce transactions with `track-ecommerce-transaction()`

Use `trackEcommerceTransaction()` to track an ecommerce transaction on the transaction level.
Arguments:

| **Argument**     | **Description**                      | **Required?** | **Type**                 |
|-----------------:|:-------------------------------------|:--------------|:-------------------------|
| `orderId`       | ID of the eCommerce transaction      | Yes           | Non-empty string         |
| `affiliation` | Transaction affiliation              | No            | String                   |
| `totalValue` | Total transaction value              | Yes           | Number                   |
| `taxValue`   | Transaction tax value                | No            | Number                   |
| `shipping`    | Delivery cost charged                | No            | Number                   |
| `city`        | Delivery address city                | No            | String                   |
| `state`       | Delivery address state               | No            | String                   |
| `country`     | Delivery address country             | No            | String                   |
| `currency`    | Currency                             | No            | String                   |
| `items`    | Array of items in the transaction       | No            | Array                    |
| `context`    | Custom context                      | No            | Array                    |
| `tstamp`         | When the transaction event occurred  | No            | Positive integer         |

The `items` argument is an array of dictionaries. Each dictionary represents one item in the transaction. These are the keys which may appear in the dictionary:

| **Field**       | **Description**                     | **Required?** | **Type**                 |
|----------------:|:------------------------------------|:--------------|:-------------------------|
| `"sku"`         | Item SKU                            | Yes           | Non-empty string         |
| `"price"`       | Item price                          | Yes           | Number                   |
| `"quantity"`    | Item quantity                       | Yes           | Int                      |
| `"name"`        | Item name                           | No            | String                   |
| `"category"`    | Item category                       | No            | String                   |
| `"context"`     | Custom context for the event        | No            | Array                    |

Example:

```js
t.trackEcommerceTransaction("order-456", null, 142, 20, 12.99, "London", null, "United Kingdom", [{
    "sku": "pbz0026",
    "price": 20,
    "quantity": 1
},
{
    "sku": "pbz0038",
    "price": 15,
    "quantity": 1
}]);
```

[Back to top](#top)


<a name="struct-event" />

### 4.5 Track structured events with `trackStructEvent()`

Use `trackStructEvent()` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument** | **Description**                                                  | **Required?** | **Type**                 |
|-------------:|:---------------------------------------------------------------  |:--------------|:-------------------------|
| `category`   | The grouping of structured events which this `action` belongs to | Yes           | Non-empty string         |
| `action`     | Defines the type of user interaction which this event involves   | Yes           | Non-empty string         |
| `label`      | A string to provide additional dimensions to the event data      | No            | String                   |
| `property`   | A string describing the object or the action performed on it     | No            | String                   |
| `value`      | A value to provide numerical data about the event                | No            | Number                   |
| `context`     | Custom context for the event                                    | No            | Array                    |
| `tstamp`     | When the structured event occurred                               | No            | Positive integer         |

Example:

```js
t.trackStructEvent("shop", "add-to-basket", null, "pcs", 2);
```

[Back to top](#top)

<a name="unstruct-event" />

### 4.6 Track unstructured events with `trackUnstructEvent()`

Use `trackUnstructEvent()` to track a custom event which consists of a name and an unstructured set of properties. This is useful when you want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow).

The arguments are as follows:

| **Argument**   | **Description**                      | **Required?** | **Type**                |
|---------------:|:-------------------------------------|:--------------|:------------------------|
| `properties`   | The properties of the event          | Yes           | JSON                    |
| `context`      | Custom context for the event         | No            | Array                   |
| `tstamp`       | When the unstructured event occurred | No            | Positive integer        |

Example:

```js
t.trackUnstructEvent({
  "schema": "iglu:com.example_company/save-game/jsonschema/1-0-2",
  "data": {
    "save_id": "4321",
    "level": 23,
    "difficultyLevel": "HARD",
    "dl_content": true
  }
})
```

The `properties` argument must be a dictionary with two fields: `schema` and `data`. `data` is a flat dictionary containing the properties of the unstructured event. `schema` identifies the JSON schema against which `data` should be validated.

For more on JSON schema, see the [blog post][self-describing-jsons].

[Back to top](#top)

[repo]: https://github.com/snowplow/snowplow-nodejs-tracker
[jsonschema]: http://snowplowanalytics.com/blog/2014/05/13/introducing-schemaver-for-semantic-versioning-of-schemas/
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
[tracker-core]: https://github.com/snowplow/snowplow/wiki/Javascript-Tracker-Core

[v1]: https://github.com/snowplow/snowplow/wiki/Node.js-Tracker-v1
