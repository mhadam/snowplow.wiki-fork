<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow technical documentation) > [**Trackers**](trackers) > Golang Tracker

## Contents

- 1. [Overview](#overview)  
- 2. [Initialization](#init)  
  - 2.1 [Import the library](#importing)
  - 2.2 [Creating a tracker](#create-tracker)
    - 2.2.1 [`RequireEmitter`](#emitter)  
    - 2.2.2 [`OptionSubject`](#subject)
    - 2.2.3 [`OptionNamespace`](#namespace)
    - 2.2.4 [`OptionAppId`](#app-id)
    - 2.2.5 [`OptionPlatform`](#platform)
    - 2.2.6 [`OptionBase64Encode`](#base64)
- 3. [Adding extra data: the Subject class](#subject-class)
  - 3.1 [`SetUserId`](#set-user-id)
  - 3.2 [`SetScreenResolution`](#set-screen-resolution)
  - 3.3 [`SetViewport`](#set-viewport-dimensions)
  - 3.4 [`SetColorDepth`](#set-color-depth)
  - 3.5 [`SetTimezone`](#set-timezone)
  - 3.6 [`SetLanguage`](#set-lang)
  - 3.7 [`SetIpAddress`](#set-ip)
  - 3.8 [`SetUseragent`](#set-useragent)
  - 3.9 [`SetDomainUserId`](#set-domain)
  - 3.10 [`SetNetworkUserId`](#set-network)
- 4. [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Custom contexts](#custom-contexts)
    - 4.1.2 [Optional timestamp argument](#tstamp-arg)
    - 4.1.3 [Optional true-timestamp argument](#true-tstamp-arg)
    - 4.1.4 [Optional event-id argument](#event-id-arg)
  - 4.2 [`TrackSelfDescribingEvent()`](#unstruct-event)
  - 4.3 [`TrackScreenView()`](#screen-view)
  - 4.4 [`TrackPageView()`](#page-view)
  - 4.5 [`TrackEcommerceTransaction()`](#ecommerce-transaction)
  - 4.6 [`TrackStructEvent()`](#struct-event)
  - 4.7 [`TrackTiming()`](#timing-event)
- 5. [Emitters](#emitter)
  - 5.1 [Under the hood](#emitter-inner-workings)

<a name="overview" />
## 1. Overview

The [Snowplow Golang Tracker](https://github.com/snowplow/snowplow-golang-tracker) allows you to track Snowplow events from your Golang apps and servers.

There are three basic types of object you will create when using the Snowplow Golang Tracker: subjects, emitters, and trackers.

A subject represents a user whose events are tracked. A tracker constructs events and sends them to an emitter. The emitter then sends the event to the endpoint you configure; a valid Snowplow Collector.

__READ ME__: As sending and processing of events is done asynchronously it is advised to create the Tracker as a singleton object.  This is due to the fact that all events are first persistently stored in a local Sqlite3 database; if multiple Trackers are created there is the possibility of duplicate sending of events and overt consumption of resources.

See [here for instructions](http://blog.ralch.com/tutorial/design-patterns/golang-singleton/) on building a Singleton in Golang.

<a name="init" />
## 2 Initialization

Assuming you have completed the [[Golang Tracker Setup]] for your project, you are now ready to initialize the Golang Tracker.

<a name="importing" />
### 2.1 Import the library

Import the Golang Tracker library like so:

```golang
import "gopkg.in/snowplow/snowplow-golang-tracker.v1/tracker"
```

You will need to refer to the package as `tracker`.  If you wish to use something shorter (or if `tracker` is already taken):

```golang
import sp "gopkg.in/snowplow/snowplow-golang-tracker.v1/tracker"
```

The package can now be referred to as `sp` rather than `tracker`.

That's it - you are now ready to initialize a tracker instance.

<a name="create-tracker" />
### 2.2 Creating a tracker

The simplest tracker initialization only requires you to provide the URI of the collector to which the tracker will log events:

```golang
import sp "gopkg.in/snowplow/snowplow-golang-tracker.v1/tracker"

emitter := sp.InitEmitter(sp.RequireCollectorUri("com.acme"))
tracker := sp.InitTracker(sp.RequireEmitter(emitter))
```

There are other optional builder functions:

| **Function Name**    | **Description**                               | **Required?** | **Default**  |
|---------------------:|:----------------------------------------------|:--------------|:-------------|
| `RequireEmitter`     | The emitter to which events are sent          | Yes           | `nil`        |
| `OptionSubject`      | The user being tracked                        | No            | `nil`        |
| `OptionNamespace`    | The name of the tracker instance              | No            | ``           |
| `OptionAppId`        | The application ID                            | No            | ``           |
| `OptionPlatform`     | The platform the Tracker is running on        | No            | `srv`        |
| `OptionBase64Encode` | Whether to enable [base 64 encoding] [base64] | No            | `true`       |

A more complete example:

```golang
subject := sp.InitSubject()
emitter := sp.InitEmitter(sp.RequireCollectorUri("com.acme"))
tracker := sp.InitTracker(
  sp.RequireEmitter(emitter),
  sp.OptionSubject(subject),
  sp.OptionNamespace("namespace"),
  sp.OptionAppId("app-id"),
  sp.OptionPlatform("mob"),
  sp.OptionBase64Encode(false),
)
```

<a name="emitter" />
#### 2.2.1 `RequireEmitter`

Accepts an argument of an Emitter instance pointer; if the object is `nil` will `panic`. See [Emitters](#emitters) for more on emitter configuration.

<a name="subject" />
#### 2.2.2 `OptionSubject`

The user which the Tracker will track. Accepts an argument of a [Subject](#subject) instance pointer. 

You don't need to set this during Tracker construction; you can use the `tracker.SetSubject()` method afterwards. In fact, you don't need to create a subject at all. If you don't, though, your events won't contain user-specific data such as timezone and language.

<a name="namespace" />
#### 2.2.3 `OptionNamespace`

If provided, the `namespace` argument will be attached to every event fired by the new tracker. This allows you to later identify which tracker fired which event if you have multiple trackers running.

<a name="app-id" />
#### 2.2.4 `OptionAppId`

The `appId` argument lets you set the application ID to any string.

<a name="platform" />
#### 2.2.5 `OptionPlatform`

By default we assume the Tracker will be running in a server environment.  To override this provide your own platform string.

<a name="base64" />
#### 2.2.6 `OptionBase64Encode`

By default, unstructured events and custom contexts are encoded into Base64 to ensure that no data is lost or corrupted. You can turn encoding on or off using the Boolean `OptionBase64Encode` function with either `true` or `false` passed in.

[Back to top](#top)

<a name="subject-class" />
## 3. Adding extra data: The Subject class

You may have additional information about your application's environment, current user and so on, which you want to send to Snowplow with each event.

Create a subject like this:

```golang
subject := sp.InitSubject()
```

The Subject class has a set of `Set...()` methods to attach extra data relating to the user to all tracked events:

* [`SetUserId`](#set-user-id)
* [`SetScreenResolution`](#set-screen-resolution)
* [`SetViewport`](#set-viewport-dimensions)
* [`SetColorDepth`](#set-color-depth)
* [`SetTimezone`](#set-timezone)
* [`SetLanguage`](#set-lang)
* [`SetIpAddress`](#set-ip)
* [`SetUseragent`](#set-useragent)
* [`SetDomainUserId`](#set-domain)
* [`SetNetworkUserId`](#set-network)

We will discuss each of these in turn below:

<a name="set-user-id" />
### 3.1 Set user ID with `SetUserId`

You can set the user ID to any string:

```golang
s.SetUserId( "{{USER ID}}" );
```

Example:

```golang
s.SetUserId("alexd");
```

[Back to top](#top)

<a name="set-screen-resolution" />
### 3.2 Set screen resolution with `SetScreenResolution`

If your code has access to the device's screen resolution, then you can pass this in to Snowplow too:

```golang
s.SetScreenResolution( {{WIDTH}}, {{HEIGHT}} );
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```golang
s.SetScreenResolution(1366, 768);
```

[Back to top](#top)

<a name="set-viewport-dimensions" />
### 3.3 Set viewport dimensions with `SetViewport`

If your code has access to the viewport dimensions, then you can pass this in to Snowplow too:

```golang
s.SetViewport( {{WIDTH}}, {{HEIGHT}} );
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```golang
s.SetViewport(300, 200);
```

[Back to top](#top)

<a name="set-color-depth" />
### 3.4 Set color depth with `SetColorDepth`

If your code has access to the bit depth of the device's color palette for displaying images, then you can pass this in to Snowplow too:

```golang
s.SetColorDepth( {{BITS PER PIXEL}} );
```

The number should be a positive integer, in bits per pixel. Example:

```golang
s.SetColorDepth(32);
```

[Back to top](#top)

<a name="set-timezone" />
### 3.5 Set timezone with `SetTimezone`

This method lets you pass a user's timezone in to Snowplow:

```golang
s.timezone( {{TIMEZONE}} );
```

The timezone should be a string:

```golang
s.SetColorDepth("Europe/London");
```

[Back to top](#top)

<a name="set-lang" />
### 3.6 Set the language with `SetLang`

This method lets you pass a user's language in to Snowplow:

```golang
s.SetLang( {{LANGUAGE}} );
```

The language should be a string:

```golang
s.SetLang('en');
```

[Back to top](#top)

<a name="set-ip" />
### 3.7 Set the IP Address with `SetIpAddress`

This method lets you pass a user's IP Address in to Snowplow:

```golang
s.SetIpAddress( {{IP ADDRESS}} );
```

The IP Address should be a string:

```golang
s.SetIpAddress('127.0.0.1');
```

[Back to top](#top)

<a name="set-useraganet" />
### 3.8 Set the useragent with `SetUseragent`

This method lets you pass a user's useragent string in to Snowplow:

```golang
s.SetUseragent( {{USERAGENT}} );
```

The useragent should be a string:

```golang
s.SetUseragent('some useragent');
```

[Back to top](#top)

<a name="set-domain" />
### 3.9 Set the Domain User ID with `SetDomainUserId`

This method lets you pass a user's Domain User ID string in to Snowplow:

```golang
s.SetDomainUserId( {{DOMAIN USER ID}} );
```

The Domain User ID should be a string:

```golang
s.SetDomainUserId('domain-uid-12');
```

[Back to top](#top)

<a name="set-network" />
### 3.10 Set the Domain User ID with `SetNetworkUserId`

This method lets you pass a user's Network User ID string in to Snowplow:

```golang
s.SetNetworkUserId( {{DOMAIN USER ID}} );
```

The Network User ID should be a string:

```golang
s.SetNetworkUserId('domain-uid-12');
```

[Back to top](#top)

<a name="events" />
## 4. Tracking specific events

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your websites and apps. We are constantly growing the range of functions available in order to capture that data more richly.

Tracking methods supported by the Golang Tracker at a glance:

| **Function**                                    | **Description**                                        |
|------------------------------------------------:|:-------------------------------------------------------|
| [`TrackPageView()`](#page-view)                 | Track and record views of web pages.                   |
| [`TrackStructEvent()`](#struct-event)           | Track a Snowplow custom structured event               |
| [`TrackSelfDescribingEvent()`](#unstruct-event) | Track a Snowplow custom unstructured event             |
| [`TrackScreenView()`](#screen-view)             | Track the user viewing a screen within the application |
| [`TrackTiming()`](#timing-event)                | Track a timing event                                   |
| [`TrackEcommerceTransaction()`](#ecommerce)     | Track an ecommerce transaction                         |

__NOTE__: All event structs require pointer values as a way of asserting properly whether or not a value has been passed that might have been required.  As such there are three functions provided that allow you to inline pointer values:

* `NewString`
* `NewInt64`
* `NewFloat64`

These all accept their respective raw value and return a pointer to this value.

<a name="common" />
### 4.1 Common

All events are tracked with specific methods on the tracker instance, of the form `TrackXXX()`, where `XXX` is the name of the event to track.

<a name="custom-contexts" />
### 4.1.1 Custom contexts

In short, custom contexts let you add additional information about the circumstances surrounding an event in the form of an array of SelfDescribingJson structs (`[]SelfDescribingJson`). Each tracking method accepts an additional optional contexts parameter, which should be a list of contexts:

```golang
contextArray := []sp.SelfDescribingJson{}
```

If a visitor arrives on a page advertising a movie, the dictionary for a single custom context in the list might look like this:


```golang
contextArray := []sp.SelfDescribingJson{
  *sp.InitSelfDescribingJson(
    "iglu:com.acme_company/movie_poster/jsonschema/2-1-1", 
    map[string]interface{}{
      "movie_name": "Solaris",
      "poster_country": "JP",
    },
  ),
}
```

This is how to fire a page view event with the above custom context:

```golang
tracker.TrackPageView(
  sp.PageViewEvent{ 
    PageUrl: sp.NewString("acme.com"),
    Contexts: contextArray,
  },
)
```

Note that even though there is only one custom context attached to the event, it still needs to be placed in an array.

<a name="tstamp-arg" />
### 4.1.2 Optional timestamp argument

Each `Track...()` method supports an optional timestamp argument; this allows you to manually override the timestamp attached to this event. The timestamp should be in milliseconds since the Unix epoch.

If you do not pass this timestamp in as an argument, then the Golang Tracker will use the current time to be the timestamp for the event.

Here is an example tracking a structured event and supplying the optional timestamp argument:

```golang
tracker.TrackStructEvent(sp.StructuredEvent{ 
  Category: sp.NewString("some category"),
  Action: sp.NewString("some action"),
  Timestamp: sp.NewInt64(1368725287000),
})
```

<a name="true-tstamp-arg" />
### 4.1.3 Optional true-timestamp argument

Each `Track...()` method supports an optional true-timestamp argument; this allows you to provide the true-timestamp attached to this event to help with the timing of events in multiple timezones. The timestamp should be in milliseconds since the Unix epoch.

Here is an example tracking a structured event and supplying the optional true-timestamp argument:

```golang
tracker.TrackStructEvent(sp.StructuredEvent{ 
  Category: sp.NewString("some category"),
  Action: sp.NewString("some action"),
  TrueTimestamp: sp.NewInt64(1368725287000),
})
```

<a name="event-id-arg" />
### 4.1.4 Optional event ID argument

Each `Track...()` method supports an optional event id argument; this allows you to manually override the event ID attached to this event. The event ID should be a valid version 4 UUID string.

Here is an example tracking a structured event and supplying the optional event ID argument:

```golang
tracker.TrackStructEvent(sp.StructuredEvent{ 
  Category: sp.NewString("some category"),
  Action: sp.NewString("some action"),
  EventId: sp.NewString("486820fb-e722-4311-b33d-d2f319b511f6"),
})
```

[Back to top](#top)

<a name="unstruct-event" />
### 4.2 Track SelfDescribing/Unstructured events with `TrackSelfDescribingEvent()`

Use `TrackSelfDescribingEvent()` to track a custom event which consists of a name and an unstructured set of properties. This is useful when:

* You want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow), or
* You want to track events which have unpredictable or frequently changing properties

The arguments are as follows:

| **Argument**    | **Description**                      | **Required?** | **Validation**          |
|----------------:|:-------------------------------------|:--------------|:------------------------|
| `Event`         | The properties of the event          | Yes           | *SelfDescribingJson     |
| `Timestamp`     | When the event occurred              | No            | *int64                  |
| `EventId`       | The event ID                         | No            | *string                 |
| `TrueTimestamp` | The true time of event               | No            | *int64                  |
| `Contexts`      | Custom contexts for the event        | No            | []SelfDescribingJson    |

Example:

```golang
// Create a data map
data := map[string]interface{}{
  "level": 5,
  "saveId": "ju302",
  "hardMode": true,
}

// Create a new SelfDescribingJson
sdj := sp.InitSelfDescribingJson("iglu:com.example_company/save-game/jsonschema/1-0-2", data)

tracker.TrackSelfDescribingEvent(sp.SelfDescribingEvent{ 
  Event: sdj,
})
```

For more on JSON schema, see the [blog post] [self-describing-jsons].

[Back to top](#top)

<a name="screen-view" />
### 4.3 Track screen views with `TrackScreenView()`

Use `TrackScreenView()` to track a user viewing a screen (or equivalent) within your app:

| **Argument**    | **Description**                     | **Required?** | **Type**                |
|----------------:|:------------------------------------|:--------------|:------------------------|
| `Name`          | Human-readable name for this screen | No            | *string                 |
| `Id`            | Unique identifier for this screen   | No            | *string                 |
| `Timestamp`     | When the event occurred             | No            | *int64                  |
| `EventId`       | The event ID                        | No            | *string                 |
| `TrueTimestamp` | The true time of event              | No            | *int64                  |
| `Contexts`      | Custom contexts for the event       | No            | []SelfDescribingJson    |

Although name and id are not individually required, at least one must be provided or the event will fail validation.

Example:

```golang
tracker.TrackScreenView(sp.ScreenViewEvent{ 
  Id: sp.NewString("Screen ID"),
})
```

[Back to top](#top)

<a name="page-view" />
### 4.4 Track pageviews with `TrackPageView()`

Use `TrackPageView()` to track a user viewing a page within your app:

| **Argument**    | **Description**                      | **Required?** | **Validation**          |
|----------------:|:-------------------------------------|:--------------|:------------------------|
| `PageUrl`       | The URL of the page                  | Yes           | *string                 |
| `PageTitle`     | The title of the page                | No            | *string                 |
| `Referrer`      | The address which linked to the page | No            | *string                 |
| `Timestamp`     | When the event occurred              | No            | *int64                  |
| `EventId`       | The event ID                         | No            | *string                 |
| `TrueTimestamp` | The true time of event               | No            | *int64                  |
| `Contexts`      | Custom contexts for the event        | No            | []SelfDescribingJson    |

Example:

```golang
tracker.TrackPageView(sp.PageViewEvent{ 
  PageUrl: sp.NewString("acme.com"),
})
```

[Back to top](#top)

<a name="ecommerce-transaction" />
### 4.5 Track ecommerce transactions with `TrackEcommerceTransaction()`

Use `TrackEcommerceTransaction()` to track an ecommerce transaction:

| **Argument**    | **Description**                 | **Required?** | **Validation**                  |
|----------------:|:--------------------------------|:--------------|:--------------------------------|
| `OrderId`       | ID of the eCommerce transaction | Yes           | *string                         |
| `TotalValue`    | Total transaction value         | Yes           | *float64                        |
| `Affiliation`   | Transaction affiliation         | No            | *string                         |
| `TaxValue`      | Transaction tax value           | No            | *float64                        |
| `Shipping`      | Delivery cost charged           | No            | *float64                        |
| `City`          | Delivery address city           | No            | *string                         |
| `State`         | Delivery address state          | No            | *string                         |
| `Country`       | Delivery address country        | No            | *string                         |
| `Currency`      | Transaction currency            | No            | *string                         |
| `Items`         | Items in the transaction        | Yes           | []EcommerceTransactionItemEvent |
| `Timestamp`     | When the event occurred         | No            | *int64                          |
| `EventId`       | The event ID                    | No            | *string                         |
| `TrueTimestamp` | The true time of event          | No            | *int64                          |
| `Contexts`      | Custom contexts for the event   | No            | []SelfDescribingJson            |

The `items` argument is an Array of TransactionItems. `TrackEcommerceTransaction` fires multiple events: one transaction event for the transaction as a whole, and one transaction item event for each element of the `Items` list. Each transaction item event will have the same timestamp, true timestamp, order ID, and currency as the main transaction event.

These are the fields with which a TransactionItem can be created.

| **Field**       | **Description**                     | **Required?** | **Validation**           |
|----------------:|:------------------------------------|:--------------|:-------------------------|
| `Sku`           | Item SKU                            | Yes           | *string                  |
| `Price`         | Item price                          | Yes           | *float64                 |
| `Quantity`      | Item quantity                       | Yes           | *int64                   |
| `Name`          | Item name                           | No            | *string                  |
| `Category`      | Item category                       | No            | *string                  |
| `EventId`       | The event ID                        | No            | *string                  |
| `Contexts`      | Custom contexts for the event       | No            | []SelfDescribingJson     |

Example of tracking a transaction containing two items:

```golang
items := []sp.EcommerceTransactionItemEvent{
  sp.EcommerceTransactionItemEvent{
    Sku: sp.NewString("pbz0026"),
    Price: sp.NewFloat64(20),
    Quantity: sp.NewInt64(1),
  },
  sp.EcommerceTransactionItemEvent{
    Sku: sp.NewString("pbz0038"),
    Price: sp.NewFloat64(15),
    Quantity: sp.NewInt64(1),
    Name: sp.NewString("red hat"),
    Category: sp.NewString("menswear"),
  },
}

tracker.TrackEcommerceTransaction(sp.EcommerceTransactionEvent{ 
  OrderId: sp.NewString("6a8078be"),
  TotalValue: sp.NewFloat64(35),
  Affiliation: sp.NewString("some-affiliation"),
  TaxValue: sp.NewFloat64(6.12),
  Shipping: sp.NewFloat64(30),
  City: sp.NewString("Dijon"),
  State: sp.NewString("Bourgogne"),
  Country: sp.NewString("France"),
  Currency: sp.NewString("EUR"),
  Items: items,
})
```

[Back to top](#top)

<a name="struct-event" />
### 4.6 Track structured events with `TrackStructEvent()`

Use `TrackStructEvent()` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument**    | **Description**                                                  | **Required?** | **Validation**       |
|----------------:|:-----------------------------------------------------------------|:--------------|:---------------------|
| `Category`      | The grouping of structured events which this `action` belongs to | Yes           | *string              |
| `Action`        | Defines the type of user interaction which this event involves   | Yes           | *string              |
| `Label`         | A string to provide additional dimensions to the event data      | No            | *string              |
| `Property`      | A string describing the object or the action performed on it     | No            | *string              |
| `Value`         | A value to provide numerical data about the event                | No            | *float64             |
| `Timestamp`     | When the event occurred                                          | No            | *int64               |
| `EventId`       | The event ID                                                     | No            | *string              |
| `TrueTimestamp` | The true time of event                                           | No            | *int64               |
| `Contexts`      | Custom contexts for the event                                    | No            | []SelfDescribingJson |

Example:

```golang
tracker.TrackStructEvent(sp.StructuredEvent{ 
  Category: sp.NewString("shop"),
  Action: sp.NewString("add-to-basket"),
  Property: sp.NewString("pcs"),
  Value: sp.NewFloat64(2),
})
```

[Back to top](#top)

<a name="timing-event" />
### 4.7 Track timing events with `TrackTiming()`

Use `TrackTiming()` to track a timing event.

The arguments are as follows:

| **Argument**    | **Description**                      | **Required?** | **Validation**          |
|----------------:|:-------------------------------------|:--------------|:------------------------|
| `Category`      | The category of the event            | Yes           | *string                 |
| `Variable`      | The variable of the event            | Yes           | *string                 |
| `Timing`        | The timing of the event              | Yes           | *int64                  |
| `Label`         | The label of the event               | No            | *string                 |
| `Timestamp`     | When the event occurred              | No            | *int64                  |
| `EventId`       | The event ID                         | No            | *string                 |
| `TrueTimestamp` | The true time of event               | No            | *int64                  |
| `Contexts`      | Custom contexts for the event        | No            | []SelfDescribingJson    |

Example:

```golang
tracker.TrackTiming(sp.TimingEvent{ 
  Category: sp.NewString("Timing Category"),
  Variable: sp.NewString("Some var"),
  Timing: sp.NewInt64(124578),
})
```

[Back to top](#top)

<a name="emitter" />
## 5. Emitters

Tracker instances must be initialized with an emitter. This section will go into more depth about the Emitter and how it works under the hood.

The simplest Emitter setup requires only the collector URI to be passed to it:

```golang
emitter := sp.InitEmitter(RequireCollectorUri("com.acme"))
```

There are other optional builder functions:

| **Function Name**     | **Description**                                | **Required?** | **Default**  |
|----------------------:|:-----------------------------------------------|:--------------|:-------------|
| `RequireCollectorUri` | The URI to send events to                      | Yes           | `nil`        |
| `OptionRequestType`   | The request type to use (GET or POST)          | No            | `POST`       |
| `OptionProtocol`      | The protocol to use (http or https)            | No            | `http`       |
| `OptionSendLimit`     | The maximum amount of events to send at a time | No            | `500`        |
| `OptionByteLimitGet`  | The byte limit when sending a GET request      | No            | `40000`      |
| `OptionByteLimitPost` | The byte limit when sending a POST request     | No            | `40000`      |
| `OptionDbName`        | Defines the path and file name of the database | No            | `events.db`  |
| `OptionCallback`      | Defines a custom callback function             | No            | `nil`        |

A more complete example:

```golang
emitter := sp.InitEmitter(
  sp.RequireCollectorUri("com.acme"),
  sp.OptionRequestType("GET"),
  sp.OptionProtocol("https"),
  sp.OptionSendLimit(50),
  sp.OptionByteLimitGet(52000),
  sp.OptionByteLimitPost(52000),
  sp.OptionDbName("/home/vagrant/test.db"),
  sp.OptionCallback(func(g []CallbackResult, b []CallbackResult)) {
    log.Println("Successes: " + IntToString(len(g)))
    log.Println("Failures: " + IntToString(len(b)))
  }),
)
```

The OptionDbName can be any valid path on your host file system (that can be created with the current user).  By default it will create the required files wherever the application is being run from.

[Back to top](#top)

<a name="emitter-inner-workings">
### 5.1 Under the hood

Once the emitter receives an event from the Tracker a few things start to happen:

* The event is added to a local Sqlite3 database (blocking execution)
* A long running go routine is started which will continue to send events as long as they can be found in the database (asynchronous)
* The emitter loop will grab a range of events from the database up until the `SendLimit` as noted __above_
* The emitter will send all of these events as determined by the Request, Protocol and ByteLimits
  - Each request is sent in its own go routine.
* Once sent it will process the results of all the requests sent and will remove all successfully sent events from the database

__IF__ all of the requests failed this loop is terminated eagerly; this is seen as a network failure so attempting to send is a waste of resources.
__IF__ there are no more events in the database the loop is terminated.

[Back to top](#top)

[jsonschema]: http://snowplowanalytics.com/blog/2014/05/13/introducing-schemaver-for-semantic-versioning-of-schemas/
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
[base64]: https://en.wikipedia.org/wiki/Base64
