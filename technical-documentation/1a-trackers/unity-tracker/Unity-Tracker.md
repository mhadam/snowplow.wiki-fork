<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Trackers**](trackers) > Unity Tracker

This page refers to version 0.1.0 of the Snowplow Unity Tracker.

## Contents

- 1. [Overview](#overview)
  - 1.1 [Snowplow Pong Demo](#demo-app)
- 2. [Initialization](#init)
  - 2.1 [Importing the library](#importing)
  - 2.2 [Creating a tracker](#create-tracker)
- 3. [Tracker](#tracker)
  - 3.1 [Constructor](#t-constructor)
  - 3.2 [Functions](#t-functions)
    - 3.2.1 [`StartEventTracking()`](#t-start)
    - 3.2.2 [`StopEventTracking()`](#t-stop)
    - 3.2.3 [`Track(IEvent)`](#t-track)
- 4. [Emitter](#emitter)
  - 4.1 [Constructor](#e-constructor)
- 5. [Subject](#subject)
  - 5.1 [`SetUserId`](#set-user-id)
  - 5.2 [`SetResolution`](#set-screen-resolution)
  - 5.3 [`SetViewPort`](#set-viewport-dimensions)
  - 5.4 [`SetColorDepth`](#set-color-depth)
  - 5.5 [`SetTimezone`](#set-timezone)
  - 5.6 [`SetLanguage`](#set-lang)
  - 5.7 [`SetIpAddress`](#set-ip-address)
  - 5.8 [`SetUseragent`](#set-useragent)
  - 5.9 [`SetNetworkUserId`](#set-network-user-id)
  - 5.10 [`SetDomainUserId`](#set-domain-user-id)
- 6. [Session](#session)
  - 6.1 [Constructor](#s-constructor)
  - 6.2 [Functions](#s-functions)
    - 6.2.1 [`SetBackground(bool)`](#s-set-background)
- 7. [Event Tracking](#event-tracking)
  - 7.1 [Events Types](#event-types)
    - 7.1.1 [`PageView`](#et-page-view)
    - 7.1.2 [`ScreenView`](#et-screen-view)
    - 7.1.3 [`Structured`](#et-structured)
    - 7.1.4 [`Timing`](#et-timing)
    - 7.1.5 [`Unstructured`](#et-unstructured)
    - 7.1.6 [`EcommerceTransaction`](#et-ecomm)
      - 7.1.6.1 [`EcommerceTransactionItem`](#et-ecomm-item)
  - 7.2 [Custom Contexts](#context-types)
    - 7.2.1 [`DesktopContext`](#ct-desktop)
    - 7.2.2 [`MobileContext`](#ct-mobile)
    - 7.2.3 [`GeoLocationContext`](#ct-geo-location)
    - 7.2.4 [`GenericContext`](#ct-generic)
  - 7.3 [`SelfDescribingJson`](#self-describing-json)
- 8. [Utilities](#utilities)
  - 8.1 [Log](#log)
  - 8.2 [ConcurrentQueue](#concurrent)

<a name="overview" />

## 1. Overview

The [Snowplow Unity Tracker](https://github.com/snowplow/snowplow-unity-tracker) allows you to track Snowplow events from your Unity games and apps.

[Back to top](#top)

<a name="demo-app" />

### 1.1 Snowplow Pong Demo

To see the Tracker in action you can download our demonstration game Snowplow Pong from [here][snowplow-pong-dl].  You will need to set a valid collector endpoint on the Settings Page to see the events being generated.

[Back to top](#top)

<a name="init" />

## 2. Initialization

Assuming you have completed the [[Unity Tracker Setup]] for your project, you are now ready to initialize the Unity Tracker.

[Back to top](#top)

<a name="importing" />

### 2.1 Importing the library

Add the following `using` lines to the top of your `.cs` scripts to access the Tracker:

```csharp
using SnowplowTracker;
using SnowplowTracker.Emitters;
using SnowplowTracker.Events;
```

You should now be able to setup the Tracker!

[Back to top](#top)

<a name="create-tracker" />

### 2.2 Creating a tracker

To instantiate a Tracker in your code (can be global or local to the process being tracked) simply instantiate the Tracker interface with the following:

```csharp
// Create Emitter and Tracker
IEmitter e1 = new AsyncEmitter ("com.collector.acme")
Tracker t1 = new Tracker(e1, "Namespace", "AppId");

// Start the Tracker
t1.StartEventTracking();
```

This is the simplest possible Tracker creation available.  For more information please review the [Tracker](#tracker) and [Emitter](#emitter) specific sections.

[Back to top](#top)

<a name="tracker" />

## 3. Tracker

The Tracker object is responsible for co-ordinating the saving and sending of events as well as managing the optional Session object.

[Back to top](#top)

<a name="t-constructor" />

### 3.1 Constructor

| **Argument Name**  | **Description**                              | **Required?**  | **Default**   |
|-------------------:|:---------------------------------------------|:---------------|:--------------|
| [`emitter`](#emitter)          | The Emitter object you create                | Yes            | Null          |
| `trackerNamespace` | The name of the tracker instance             | Yes            | Null          |
| `appId`            | The application ID                           | Yes            | Null          |
| [`subject`](#subject)          | The Subject that defines a user              | No             | Null          |
| [`session`](#session)          | The Session object you create                | No             | Null          |
| `platform`         | The device the Tracker is running on         | No             | Mobile        |
| `base64Encoded`    | If we [base 64 encode][base64] json values   | No             | True          |

A full Tracker construction should look like the following:

```csharp
IEmitter e1 = new AsyncEmitter ("com.collector.acme")
Subject subject = new Subject();
Session session = new Session();
Tracker t1 = new Tracker(e1, "Namespace", "AppId", subject, session, DevicePlatforms.Desktop, true);
```

All of these variables can be altered after creation with the accompanying `tracker.SetXXX()` function.

[Back to top](#top)

<a name="t-functions" />

### 3.2 Functions

The Tracker also contains several critical functions that must be used to start Tracking.

[Back to top](#top)

<a name="t-start" />

#### 3.2.1 `StartEventTracking()`

This function must be called before any events will start being stored or sent.  This is due to the fact that we do not want to start any background processing from the constructors so it is left up to the developer to choose when to start everything up.

This function:

* Starts the background emitter thread
* Starts the background event processor thread
* Starts the background session check timer (Optional)

Once this is run everything should be in place for asynchronous event tracking.

[Back to top](#top)

<a name="t-stop" />

#### 3.2.2 `StopEventTracking()`

If you need to temporarily halt the Tracker from tracking events you can run this function.  This will bring all event processing, sending and collection to a halt and nothing will be started again until `StartEventTracking()` is fired again.

[Back to top](#top)

<a name="t-track" />

#### 3.2.3 `Track(IEvent)`

This is the function used for Tracking all events.  You can pass any of [these](#event-types) event types to this function.

```csharp
tracker.Track(IEvent newEvent);
```

[Back to top](#top)

<a name="emitter" />

## 4. Emitter

The Emitter object is responsible for sending and storing all events.

We have two emitters available currently:

* `SyncEmitter` : Slow blocking synchronous operation, useful for testing but should not be used in production.
* `AsyncEmitter` : Fully asynchronous operation which uses the ThreadPool to perform all of its operations.

[Back to top](#top)

<a name="e-constructor" />

### 4.1 Constructor

| **Argument Name**  | **Description**                              | **Required?**  | **Default**   |
|-------------------:|:---------------------------------------------|:---------------|:--------------|
| `endpoint`         | The collector uri the emitter should use     | Yes            | Null          |
| `protocol`         | The request Protocol (HTTP or HTTPS)         | No             | HTTP          |
| `method`           | The HTTP Method (GET or POST)                | No             | POST          |
| `sendLimit`        | The amount of events to send at a time       | No             | 500           |
| `byteLimitGet`     | The byte limit for a GET request             | No             | 52000         |
| `byteLimitPost`    | The byte limit for a POST request            | No             | 52000         |

A full Emitter construction should look like the following:

```csharp
IEmitter e1 = new AsyncEmitter ("com.collector.acme", HttpProtocol.HTTPS, HttpMethod.GET, 50, 30000, 30000);
```

All of these variables can be altered after creation with the accompanying `emitter.SetXXX()` function.  However do be aware that multiple threads will be accessing these variables so to be safe always shut the Tracker down using `StopEventTracking()` before ammending anything.

**NOTE:** Be aware that when sending events via `GET` all events will be sent individually.  This means that if your `sendLimit` is 500 there is the potential for 500 Threads to be spawned at the same time which can cause serious performance issues.  To alleviate this concern simply drop your `sendLimit` to a more manageable range.

[Back to top](#top)

<a name="subject" />

## 5. Subject

You may have additional information about your application's environment, current user and so on, which you want to send to Snowplow with each event.

The Subject class has a set of `Set...()` methods to attach extra data relating to the user to all tracked events:

* [`SetUserId`](#set-user-id)
* [`SetScreenResolution`](#set-screen-resolution)
* [`SetViewport`](#set-viewport-dimensions)
* [`SetColorDepth`](#set-color-depth)
* [`SetTimezone`](#set-timezone)
* [`SetLanguage`](#set-lang)
* [`SetIpAddress`](#set-ip-address)
* [`SetUseragent`](#set-useragent)
* [`SetNetworkUserId`](#set-network-user-id)
* [`SetDomainUserId`](#set-domain-user-id)

Here are some examples:

```csharp
Subject s1 = new Subject();
s1.SetUserId("Kevin Gleason");
s1.SetLanguage("en-gb");
s1.SetScreenResolution(1920, 1080);
```

After that, you can add your Subject to your Tracker like so:

```csharp
Tracker t1 = new Tracker(..., s1)

// OR

t1.SetSubject(s1);
```

[Back to top](#top)

<a name="set-user-id" />

### 5.1 Set user ID with `SetUserId`

You can set the user ID to any string:

```csharp
s1.SetUserId( "{{USER ID}}" )
```

Example:

```csharp
s1.SetUserId("alexd")
```

[Back to top](#top)

<a name="set-screen-resolution" />

### 5.2 Set screen resolution with `SetScreenResolution`

If your C# code has access to the device's screen resolution, then you can pass this in to Snowplow too:

```csharp
t1.SetScreenResolution( {{WIDTH}}, {{HEIGHT}} )
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```csharp
t1.SetScreenResolution(1366, 768)
```

[Back to top](#top)

<a name="set-viewport-dimensions" />

### 5.3 Set viewport dimensions with `SetViewport`

If your C# code has access to the viewport dimensions, then you can pass this in to Snowplow too:

```csharp
s.SetViewport( {{WIDTH}}, {{HEIGHT}} )
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```csharp
s.SetViewport(300, 200)
```

[Back to top](#top)

<a name="set-color-depth" />

### 5.4 Set color depth with `SetColorDepth`

If your C# code has access to the bit depth of the device's color palette for displaying images, then you can pass this in to Snowplow too:

```csharp
s.SetColorDepth( {{BITS PER PIXEL}} )
```

The number should be a positive integer, in bits per pixel. Example:

```csharp
s.SetColorDepth(32)
```

[Back to top](#top)

<a name="set-timezone" />

### 5.5 Set timezone with `SetTimezone`

This method lets you pass a user's timezone in to Snowplow:

```csharp
s.SetTimezone( {{TIMEZONE}} )
```

The timezone should be a string:

```csharp
s.SetTimezone("Europe/London")
```

[Back to top](#top)

<a name="set-lang" />

### 5.6 Set the language with `SetLanguage`

This method lets you pass a user's language in to Snowplow:

```csharp
s.SetLanguage( {{LANGUAGE}} )
```

The language should be a string:

```csharp
s.SetLanguage('en')
```

[Back to top](#top)

<a name="set-ip-address" />

### 5.7 `SetIpAddress`

This method lets you pass a user's IP Address in to Snowplow:

```csharp
SetIpAddress( {{IP ADDRESS}} )
```

The IP address should be a string:

```csharp
subj.SetIpAddress("127.0.0.1");
```

[Back to top](#top)

<a name="set-useragent" />

### 5.8 `SetUseragent`

This method lets you pass a useragent in to Snowplow:

```csharp
SetUseragent( {{USERAGENT}} )
```

The useragent should be a string:

```csharp
subj.SetUseragent("Agent Smith");
```

[Back to top](#top)

<a name="set-network-user-id" />

### 5.9 `SetNetworkUserId`

This method lets you pass a Network User ID in to Snowplow:

```csharp
SetNetworkUserId( {{NUID}} )
```

The network user id should be a string:

```csharp
subj.SetNetworkUserId("network-id");
```

[Back to top](#top)

<a name="set-domain-user-id" />

### 5.10 `SetDomainUserId`

This method lets you pass a Domain User ID in to Snowplow:

```csharp
SetDomainUserId( {{DUID}} )
```

The domain user id should be a string:

```csharp
subj.SetDomainUserId("domain-id");
```

[Back to top](#top)

<a name="session" />

## 6. Session

The Session object is responsible for maintaining persistent data around user sessions over the life-time of an application.

[Back to top](#top)

<a name="s-constructor" />

### 6.1 Constructor

| **Argument Name**   | **Description**                              | **Required?**  | **Default**   |
|--------------------:|:---------------------------------------------|:---------------|:--------------|
| `foregroundTimeout` | The time until a session expires in focus    | No             | 600 (s)       |
| `backgroundTimeout` | The time until a session expires in back     | No             | 300 (s)       |
| `checkInterval`     | How often to validate the session timeout    | No             | 15 (s)        |

A full Session construction should look like the following:

```csharp
Session session = new Session (1200, 600, 30);
```

The timeout's refer to the length of time the session remains active after the last event is sent.  As long as events are sent within this limit the session will not timeout.

[Back to top](#top)

<a name="s-functions" />

### 6.2 Functions

[Back to top](#top)

<a name="s-set-background" />

#### 6.2.1 `SetBackground(bool)`

Will set whether or not the application is in the background.  It is up to the developer to set this metric if they wish to have a different timeout for foreground and background.

[Back to top](#top)

<a name="event-tracking" />

## 7. Event Tracking

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your websites and apps. We are constantly growing the range of functions available in order to capture that data more richly.

Events supported by the Unity Tracker at a glance:

| **Events**                                                    | **Description*                                         |
|--------------------------------------------------------------:|:-------------------------------------------------------|
| [`Track(PageView)`](#et-page-view)                            | Track and record views of web pages                    |
| [`Track(ScreenView)`](#et-screen-view)                        | Track the user viewing a screen within the application |
| [`Track(Structured)`](#et-structured)                         | Track a Snowplow custom structured event               |
| [`Track(Timing)`](#et-timing)                                 | Track a Timing with Category event                     |
| [`Track(Unstructured)`](#et-unstructured)                     | Track a Snowplow custom unstructured event             |
| [`Track(EcommerceTransaction)`](#et-ecomm)                    | Track an ecommerce transaction and its items           |

[Back to top](#top)

<a name="event-types" />

## 7. Event Types

[Back to top](#top)

<a name="et-page-view" />

#### 7.1.1 Track page views with `Track(PageView)`

You can use `Track(PageView)` to track a user viewing a web page within your app.

Arguments are:

| **Argument**    | **Description**                      | **Required?** | **Type**                   |
|----------------:|:-------------------------------------|:--------------|:---------------------------|
| `pageUrl`       | The URL of the page                  | Yes           | `string`                   |
| `pageTitle`     | The title of the page                | No            | `string`                   |
| `referrer`      | The address which linked to the page | No            | `string`                   |
| `customContexts`| Optional custom context              | No            | `List<IContext>`           |
| `timestamp`     | Optional timestamp                   | No            | `long`                     |
| `eventId`       | Optional custom event id             | No            | `string`                   |

Examples:

```csharp
t1.Track(new PageView()
    .SetPageUrl("www.example.com")
    .SetPageTitle("example")
    .SetReferrer("www.referrer.com")
    .Build());

t1.Track(new PageView()
    .SetPageUrl("www.example.com")
    .SetPageTitle("example")
    .SetReferrer("www.referrer.com")
    .SetCustomContext(contextList)
    .SetTimestamp(1423583655000)
    .SetEventId("uid-1")
    .Build());
```

[Back to top](#top)

<a name="et-screen-view" />

#### 7.1.2 Track screen views with `Track(ScreenView)`

Use `Track(ScreenView)` to track a user viewing a screen (or equivalent) within your app. You **must** use either `name` or `id`. Arguments are:

| **Argument**    | **Description**                     | **Required?** | **Type**                   |
|----------------:|:------------------------------------|:--------------|:---------------------------|
| `name`          | Human-readable name for this screen | No            | `string`                   |
| `id`            | Unique identifier for this screen   | No            | `string`                   |
| `customContexts`| Optional custom context             | No            | `List<IContext>`           |
| `timestamp`     | Optional timestamp                  | No            | `long`                     |
| `eventId`       | Optional custom event id            | No            | `string`                   |

Examples:

```csharp
t1.Track(new ScreenView()
    .SetName("HUD > Save Game")
    .SetId("screen23")
    .Build());

t1.Track(new ScreenView()
    .SetName("HUD > Save Game")
    .SetId("screen23")
    .SetCustomContext(contextList)
    .SetTimestamp(1423583655000)
    .SetEventId("uid-1")
    .Build());
```

[Back to top](#top)

<a name="et-structured" />

#### 7.1.3 Track structured events with `Track(Structured)`

Use `Track(Structured)` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument**    | **Description**                                                  | **Required?** | **Type**                   |
|----------------:|:---------------------------------------------------------------  |:--------------|:---------------------------|
| `category`      | The grouping of structured events which this `action` belongs to | Yes           | `string`                   |
| `action`        | Defines the type of user interaction which this event involves   | Yes           | `string`                   |
| `label`         | A string to provide additional dimensions to the event data      | No            | `string`                   |
| `property`      | A string describing the object or the action performed on it     | No            | `string`                   |
| `value`         | A value to provide numerical data about the event                | No            | `double`                   |
| `customContexts`| Optional custom context                                          | No            | `List<IContext>`           |
| `timestamp`     | Optional timestamp                                               | No            | `long`                     |
| `eventId`       | Optional custom event id                                         | No            | `string`                   |

Examples:

```csharp
t1.Track(new Structured()
    .SetCategory("shop")
    .SetAction("add-to-basket")
    .Build());

t1.Track(new Structured()
    .SetCategory("shop")
    .SetAction("add-to-basket")
    .SetLabel("Add To Basket")
    .SetProperty("pcs")
    .SetValue(2.00)
    .SetCustomContext(contextList)
    .SetTimestamp(1423583655000)
    .SetEventId("uid-1")
    .Build());
```

[Back to top](#top)

<a name="et-timing" />

#### 7.1.4 Track timing events with `Track(Timing)`

Use `Track(Timing)` to track an event related to a custom timing.

| **Argument**    | **Description**                                 | **Required?** | **Type**                   |
|----------------:|:------------------------------------------------|:--------------|:---------------------------|
| `category`      | The category of the timed event                 | Yes           | `string`                   |
| `label`         | The label of the timed event                    | No            | `string`                   |
| `timing`        | The timing measurement in milliseconds          | Yes           | `int`                      |
| `variable`      | The name of the timed event                     | Yes           | `string`                   |
| `customContexts`| Optional custom context                         | No            | `List<IContext>`           |
| `timestamp`     | Optional timestamp                              | No            | `long`                     |
| `eventId`       | Optional custom event id                        | No            | `string`                   |

Examples:

```csharp
t1.Track(new Timing()
    .SetCategory("category")
    .SetVariable("variable")
    .SetTiming(1)
    .Build());

t1.Track(new Timing()
    .SetCategory("category")
    .SetVariable("variable")
    .SetTiming(1)
    .SetLabel("label")
    .SetCustomContext(contextList)
    .SetTimestamp(1423583655000)
    .SetEventId("uid-1")
    .Build());
```

[Back to top](#top)

<a name="et-unstructured" />

#### 7.1.5 Track unstructured events with `Track(Unstructured)`

Custom unstructured events are a flexible tool that enable Snowplow users to define their own event types and send them into Snowplow.

When a user sends in a custom unstructured event, they do so as a JSON of name-value properties, that conforms to a JSON schema defined for the event earlier.

Use `Track(Unstructured)` to track a custom event which consists of a name and an unstructured set of properties. This is useful when:

* You want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow), or
* You want to track events which have unpredictable or frequently changing properties

The arguments are as follows:

| **Argument**    | **Description**                   |  **Required?** |  **Type**                  |
|----------------:|:----------------------------------|:---------------|:---------------------------|
| `eventData`     | The properties of the event       | Yes            | [`SelfDescribingJson`](#self-describing-json)       |
| `customContexts`| Optional custom context           | No             | `List<IContext>`           |
| `timestamp`     | Optional timestamp                | No             | `long`                     |
| `eventId`       | Optional custom event id          | No             | `string`                   |

Example event json to track:

```json
{
  "schema": "iglu:com.acme/save_game/jsonschema/1-0-0",
  "data": {
    "levelName": "Barrels o' Fun",
    "levelIndex": 23
  }
}
```

How to set it up?

```csharp
// Create a Dictionary of your event data
Dictionary<string, object> eventDict = new Dictionary<string, object>();
eventDict.Add("levelName", "Barrels o' Fun");
eventDict.Add("levelIndex", 23);

// Create your event data
SelfDescribingJson eventData = new SelfDescribingJson("iglu:com.acme/save_game/jsonschema/1-0-0", eventDict);

// Track your event with your custom event data
t1.Track(new Unstructured()
    .SetEventData(eventData)
    .Build();

// OR

t1.Track(new Unstructured()
    .SetEventData(eventData)
    .SetCustomContext(contextList)
    .SetTimestamp(1423583655000)
    .SetEventId("uid-1")
    .Build();
```

For more on JSON schema, see the [blog post][self-describing-jsons].

[Back to top](#top)

<a name="et-ecomm" />

#### 7.1.6 Track ecommerce transactions with `Track(EcommerceTransaction)`

Use `Track(EcommerceTransaction)` to track an ecommerce transaction.

Arguments:

| **Argument**    | **Description**                      | **Required?** | **Type**                   |
|----------------:|:-------------------------------------|:--------------|:---------------------------|
| `orderId`       | ID of the eCommerce transaction      | Yes           | `string`                   |
| `totalValue`    | Total transaction value              | Yes           | `double`                   |
| `affiliation`   | Transaction affiliation              | No            | `string`                   |
| `taxValue`      | Transaction tax value                | No            | `double`                   |
| `shipping`      | Delivery cost charged                | No            | `double`                   |
| `city`          | Delivery address city                | No            | `string`                   |
| `state`         | Delivery address state               | No            | `string`                   |
| `country`       | Delivery address country             | No            | `string`                   |
| `currency`      | Transaction currency                 | No            | `string`                   |
| `items`         | Items in the transaction             | Yes           | `List<EcommerceTransactionItem>` |
| `customContexts`| Optional custom context              | No            | `List<IContext>`           |
| `timestamp`     | Optional timestamp                   | No            | `long`                     |
| `eventId`       | Optional custom event id             | No            | `string`                   |

The `items` argument is a `List` of individual `EcommerceTransactionItem` elements representing the items in the e-commerce transaction. Note that `Track(EcommerceTransaction)` fires multiple events: one transaction event for the transaction as a whole, and one transaction item event for each element of the `items` `List`.

Each transaction item event will have the same timestamp, orderId, and currency as the main transaction event.

[Back to top](#top)

<a name="et-ecomm-item" />

#### 7.1.6.1 `EcommerceTransactionItem`

To instantiate a `EcommerceTransactionItem` in your code, simply use the following constructor signature:

```csharp
EcommerceTransactionItem item = new EcommerceTransactionItem ()
      .SetSku ("sku")
      .SetPrice (10.2)
      .SetQuantity (1)
      .SetName ("name")
      .SetCategory ("category")
      .Build ()
```

These are the fields that can appear as elements in each `EcommerceTransactionItem` element of the transaction item's `List`:

| **Field**       | **Description**                      | **Required?** | **Type**                   |
|----------------:|:-------------------------------------|:--------------|:---------------------------|
| `sku`           | Item SKU                             | Yes           | `string`                   |
| `price`         | Item price                           | Yes           | `double`                   |
| `quantity`      | Item quantity                        | Yes           | `int`                      |
| `name`          | Item name                            | No            | `string`                   |
| `category`      | Item category                        | No            | `string`                   |
| `customContexts`| Optional custom context              | No            | `List<IContext>`           |
| `eventId`       | Optional custom event id             | No            | `string`                   |

Example of tracking a transaction containing two items:

```csharp

// Create some Transaction Items
EcommerceTransactionItem item1 = new EcommerceTransactionItem ()
    .SetSku ("item_sku_1")
    .SetPrice (10.2)
    .SetQuantity (1)
    .SetName ("item_name_1")
    .SetCategory ("item_category")
    .Build ();

EcommerceTransactionItem item2 = new EcommerceTransactionItem()
    .SetSku("item_sku_2")
    .SetPrice(1.00)
    .SetQuantity(1)
    .SetName("item_name_2")
    .SetCategory("item_category")
    .Build();

// Add these items to a List
List<EcommerceTransactionItem> items = new List<EcommerceTransactionItem>();
items.Add(item1);
items.Add(item2);

// Now Track the Transaction by using this list of items as an argument
tracker.Track(new EcommerceTransaction()
    .SetOrderId("order_id_1")
    .SetTotalValue(300.00)
    .SetAffiliation("my_affiliate")
    .SetTaxValue(30.00)
    .SetShipping(10.00)
    .SetCity("Boston")
    .SetState("Massachusetts")
    .SetCountry("USA")
    .SetCurrency("USD")
    .SetItems(items)
    .Build());
```

[Back to top](#top)

<a name="context-types" />

### 7.2 Custom Contexts

Custom contexts are Self Describing Jsons with extra descriptive information that can be optionally attached to any Snowplow event with `SetCustomContexts(...)`.
We provide several builders for Snowplow custom contexts as well as a generic builder if you wish to define and send your own custom context!

For ease of development you are also able to extend the `IContext` interface or the `AbstractContext` class for your own contexts if you so wish.

All of these contexts will need to be combined into a `List<IContext>` before being attachable to Snowplow Events.

[Back to top](#top)

<a name="ct-desktop" />

#### 7.2.1 `DesktopContext`

The following arguments can be used in a DesktopContext:

| **Field**            | **Description**                      | **Required?** | **Type**                   |
|---------------------:|:-------------------------------------|:--------------|:---------------------------|
| `osType`             | The Operating System Type            | Yes           | `string`                   |
| `osVersion`          | The Version of the Operating System  | Yes           | `string`                   |
| `osServicePack`      | Service Pack information             | No            | `string`                   |
| `osIs64Bit`          | If the OS is 32 or 64 bit            | No            | `bool`                     |
| `deviceManufacturer` | Who made the device                  | No            | `string`                   |
| `deviceModel`        | What is the device model             | No            | `string`                   |
| `processorCount`     | How many cores does the device have  | No            | `int`                      |

An example of a DesktopContext construction:

```csharp
DesktopContext context = new DesktopContext ()
    .SetOsType("OS-X")
    .SetOsVersion("10.10.5")
    .SetOsServicePack("Yosemite")
    .SetOsIs64Bit(true)
    .SetDeviceManufacturer("Apple")
    .SetDeviceModel("Macbook Pro")
    .SetDeviceProcessorCount(4)
    .Build ();
```

[Back to top](#top)

<a name="ct-mobile" />

#### 7.2.2 `MobileContext`

The following arguments can be used in a MobileContext:

| **Field**            | **Description**                      | **Required?** | **Type**                   |
|---------------------:|:-------------------------------------|:--------------|:---------------------------|
| `osType`             | The Operating System Type            | Yes           | `string`                   |
| `osVersion`          | The Version of the Operating System  | Yes           | `string`                   |
| `deviceManufacturer` | Who made the device                  | Yes           | `string`                   |
| `deviceModel`        | What is the device model             | Yes           | `string`                   |
| `carrier`            | The name of the carrier              | No            | `string`                   |
| `networkType`        | The type of network                  | No            | `NetworkType`              |
| `networkTechnology`  | The networks technlogy               | No            | `string`                   |
| `openIdfa`           | An OpenIDFA UUID                     | No            | `string`                   |
| `appleIdfa`          | An Apple IDFA UUID                   | No            | `string`                   |
| `appleIdfv`          | An Apple IDFV UUID                   | No            | `string`                   |
| `androidIdfa`        | An Android IDFA UUID                 | No            | `string`                   |

An example of a MobileContext construction:

```csharp
MobileContext context = new MobileContext ()
    .SetOsType("iOS")
    .SetOsVersion("9.0")
    .SetDeviceManufacturer("Apple")
    .SetDeviceModel("iPhone 6S+")
    .SetCarrier("FREE")
    .SetNetworkType(NetworkType.Mobile)
    .SetNetworkTechnology("LTE")
    .Build ();
```

[Back to top](#top)

<a name="ct-geo-location" />

#### 7.2.3 `GeoLocationContext`

The following arguments can be used in a GeoLocationContext:

| **Field**                   | **Description**                      | **Required?** | **Type**                   |
|----------------------------:|:-------------------------------------|:--------------|:---------------------------|
| `latitude`                  | The user latitude                    | Yes           | `double`                   |
| `longitude`                 | The user longitude                   | Yes           | `double`                   |
| `latitudeLongitudeAccuracy` | The user lat-long accuracy           | No            | `double`                   |
| `altitude`                  | The user altitude                    | No            | `double`                   |
| `altitudeAccuracy`          | The user alt accuracy                | No            | `double`                   |
| `bearing`                   | The user bearing                     | No            | `double`                   |
| `speed`                     | The user speed                       | No            | `double`                   |
| `timestamp`                 | A timestamp in ms                    | No            | `long`                     |

An example of a GeoLocationContext construction:

```csharp
GeoLocationContext context = new GeoLocationContext ()
    .SetLatitude(123.564)
    .SetLongitude(-12.6)
    .SetLatitudeLongitudeAccuracy(5.6)
    .SetAltitude(5.5)
    .SetAltitudeAccuracy(2.1)
    .SetBearing(3.2)
    .SetSpeed(100.2)
    .SetTimestamp(1234567890000)
    .Build ();
```

[Back to top](#top)

<a name="ct-generic" />

#### 7.2.4 `GenericContext`

The GenericContext is a simple builder with three functions:

* `SetSchema(string)` : Sets the Context Schema Path
* `Add(string, object)` : Adds a single key-pair value to the data packet of this context
* `AddDict(string, object)` : Adds a dictionary of key-pair values to the data packet

You must set a schema string or a RuntimeException will be thrown.

An example of a GenericContext construction:

```csharp
GenericContext context = new GenericContext()
    .SetSchema("iglu:com.acme/acme_context/jsonschema/1-0-0")
    .Add("context", "custom")
    .Build();
```

[Back to top](#top)

<a name="self-describing-json" />

### 7.3 SelfDescribingJson

A `SelfDescribingJson` is used as a wrapper around a `Dictionary<string, object>`.  After creating the Dictionary you want to wrap you can create a `SelfDescribingJson` using the following:

```csharp
// Data as a Dictionary
Dictionary<string, object> data = new Dictionary<string, object>();
data.Add("Event", "Data")

// We then create a new SelfDescribingJson
SelfDescribingJson json = new SelfDescribingJson("iglu:com.acme/example/jsonschema/1-0-0", data);
```

This object is now ready to be Tracked within an Unstructured Event.

You can create a SelfDescribingJson with the following arguments:

| **Argument** | **Description**                           | **Required?** | **Type**                    |
|-------------:|:------------------------------------------|:--------------|:----------------------------|
| `schema`     | JsonSchema that describes the data        | Yes           | `string`                    |
| `data`       | Data that will be validated by the schema | No            | `Dictionary<string,object`> |

[Back to top](#top)

<a name="utilities" />

## 8. Utilities

The Tracker also provides several extra utilities that can be used.

[Back to top](#top)

<a name="log" />

### 8.1 Log

There is a custom logging wrapper which allows you to control the level of Tracker logging that occurs. You can set the level via:

```csharp
Log.SetLogLevel({0,1,2 or 3});
```

* `0` : Turns of all logging
* `1` : Error only
* `2` : Error and Debug
* `3` : Error, Debug and Verbose

[Back to top](#top)

<a name="concurrent" />

### 8.2 ConcurrentQueue

Due to the .NET 2.0 limitation we have had to implement our own ThreadSafe queue which can be found in the following package:

* `SnowplowTracker.Collections`

[Back to top](#top)

[snowplow-pong-dl]: http://
[base64]: https://en.wikipedia.org/wiki/Base64
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
