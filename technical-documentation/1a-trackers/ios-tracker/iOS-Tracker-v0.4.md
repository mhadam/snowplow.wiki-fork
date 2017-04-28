<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Trackers**](trackers) > iOS Tracker

This page refers to version 0.4.0 of the Snowplow Objective-C Tracker, which is the latest version. Documentation for earlier versions is available:

* *[Version 0.3][ios-0.3]*
* *[Version 0.2][ios-0.2]*
* *[Version 0.1][ios-0.1]*

## Contents

- 1. [Overview](#overview)
  - 1.1 [Demonstration App](#demo-app)
- 2. [Initialization](#init)
  - 2.1 [Importing the library](#importing)
  - 2.2 [Creating a tracker](#create-tracker)
    - 2.2.1 [`SnowplowEmitter`](#emitter)
    - 2.2.2 [`namespace`](#namespace)
    - 2.2.3 [`appId`](#app-id)
    - 2.2.4 [`base64Encoded`](#base64)
- 3. [Adding extra data](#add-data)
  - 3.1 [`setUserId`](#set-user-id)
  - 3.2 [Sending IFA](#sending-ifa)
- 4. [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Custom contexts](#custom-contexts)
    - 4.1.2 [Optional timestamp argument](#tstamp-arg)
    - 4.1.3 [Tracker method return values](#return-values)
  - 4.2 [`trackScreenView:`](#screen-view)
  - 4.3 [`trackPageView:`](#page-view)
  - 4.4 [`trackEcommerceTransaction:`](#ecommerce-transaction)
  - 4.5 [`trackStructuredEvent:`](#struct-event)
  - 4.6 [`trackUnstructuredEvent:`](#unstruct-event)
  - 4.7 [`trackTimingWithCategory:`](#timing)
- 5. [Sending events: `SnowplowEmitter`](#emitters)
  - 5.1 [Using a buffer](#buffer)
  - 5.2 [Choosing the HTTP method](#http-method)
  - 5.3 [Emitter Callback](#emitter-callback)
  - 5.4 [Sending HTTP requests](#http-request)

<a name="overview" />

## 1. Overview

The [Snowplow iOS Tracker](https://github.com/snowplow/snowplow-ios-tracker) allows you to track Snowplow events from your iOS apps and games. It supports iOS 6.0+.

The tracker should be straightforward to use if you are comfortable with iOS development; its API is modelled after Snowplow's [[Python Tracker]] so any prior experience with that tracker is helpful but not necessary. If you haven't already, have a look at the [[iOS Tracker Setup]] guide before continuing.

You can also find detailed documentation for the method calls in the tracker classes available as part of the [CocoaPods documentation](http://cocoadocs.org/docsets/SnowplowTracker/).

[Back to top](#top)

<a name="demo-app" />

### 1.1 Demonstration App

If you would like to see the Tracker in action you can launch the demo app like so:

* Download the github repo: `git clone https://github.com/snowplow/snowplow-objc-tracker.git`
* In XCode open the `SnowplowDemo.xcworkspace` file.
* Select the device you want to launch the SnowplowDemo into!

You will then need to simply enter a valid endpoint URL and hit the `Start Demo!` button.

[Back to top](#top)

<a name="init" />

## 2. Initialization

Assuming you have completed the [[iOS Tracker Setup]] for your project, you are now ready to initialze the Snowplow Tracker.

<a name="importing" />

## 2.1 Importing the library

Adding the library into your project is as simple as adding the headers into your class file:

```objective-c
#import <SnowplowTracker.h>
#import <SnowplowEmitter.h>
```

If you have manually copied the library into your project, don't forget to change your import syntax:

```objective-c
#import "SnowplowTracker.h"
#import "SnowplowEmitter.h"
```

If you have statically added the library you will need to further amend your syntax:

```objective-c
#import "SnowplowTracker/SnowplowTracker.h"
#import "SnowplowTracker/SnowplowEmitter.h"
```

That's it - you are now ready to initialize a tracker instance.

[Back to top](#top)

<a name="create-tracker" />

### 2.2 Creating a tracker

To instantiate a tracker in your code simply instantiate the `SnowplowTracker` class with the constructor:
```objective-c
- (id) initWithCollector:(SnowplowEmitter *)collector_
                   appId:(NSString *)appId_
           base64Encoded:(Boolean)encoded
               namespace:(NSString *)namespace_
```

For example:
```objective-c
SnowplowTracker *t1 = [[SnowplowTracker alloc] initWithCollector:collector appId:@"AF003" base64Encoded:false namespace:@"cloudfront"];
```

| **Argument Name** | **Description**                              |
|------------------:|:---------------------------------------------|
| `collector`       | The SnowplowEmitter object you create        |
| `namespace`       | The name of the tracker instance             |
| `appId`           | The application ID                           |
| `base64Encoded`   | Whether to enable [base 64 encoding][base64] |

[Back to top](#top)

<a name="emitter" />

#### 2.2.1 `collector`

This is a single `SnowplowEmitter` object that will be used to send all the tracking events created by the `SnowplowTracker` to a collector. See [Sending events](#emitters) for more on its configuration.

<a name="namespace" />

#### 2.2.2 `namespace`

If provided, the `namespace` argument will be attached to every event fired by the new tracker. This allows you to later identify which tracker fired which event if you have multiple trackers running.

<a name="app-id" />

#### 2.2.3 `appId`

The `appId` argument lets you set the application ID to any string.

<a name="base64" />

#### 2.2.4 `base64Encoded`

By default, unstructured events and custom contexts are encoded into Base64 to ensure that no data is lost or corrupted. You can turn encoding on or off using the Boolean `base64Encoded` argument.

[Back to top](#top)

<a name="add-data" />

## 3. Adding extra data

Unlike the other Trackers, the iOS tracker automatically collects your platform, screen resolution, viewport, color depth, timezone and language from the device. You can still however, set your user ID to properly track different users if you require it.

* [`setUserId`](#set-user-id)
* [Sending IFA](#sending-ifa)

[Back to top](#top)

<a name="set-user-id" />

### 3.1 Set user ID with `setUserId`

You can set the user ID to any string:

```objective-c
s1.setUserId( "{{USER ID}}" )
```

Example:

```objective-c
[tracker setUserId:@"alexd"];
```

<a name="sending-ifa" />

### 3.2 Sending IFA

Apps that do not display advertisements are not allowed to access Apple's Identifier For Advertisers (IFA). For this reason, the Snowplow iOS Tracker will only send IFA as part of the `mobile_context` which is attached to each event **if** you have the `AdSupport.framework` included in your app (and are therefore intending to serve ads).

For the avoidance of doubt, you can also avoid sending IFA regardless of your advertising situation, thus:

* Click on **Build Settings** to your app's project in Xcode
* Search for **Preprocessor Macros**
* Add a macro defined as`SNOWPLOW_NO_IFA = 1`

<a name="events" />

## 4. Tracking specific events

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your websites and apps. We are constantly growing the range of functions available in order to capture that data more richly.

Tracking methods supported by the iOS Tracker at a glance:

| **Function**                                           | **Description*                                         |
|-------------------------------------------------------:|:-------------------------------------------------------|
| [`trackScreenView:`](#screen-view)                     | Track the user viewing a screen within the application |
| [`trackPageView:`](#page-view)                         | Track and record views of web pages.                   |
| [`trackEcommerceTransaction:`](#ecommerce-transaction) | Track an ecommerce transaction and its items           |
| [`trackStructuredEvent:`](#struct-event)               | Track a Snowplow custom structured event               |
| [`trackUnstructuredEvent:`](#unstruct-event)           | Track a Snowplow custom unstructured event             |
| [`trackTiming:`](#timing)                              | Track a Snowplow user timing event                     |

[Back to top](#top)
<a name="common" />

### 4.1 Common

All events are tracked with specific methods on the tracker instance, of the form `trackXXX()`, where `XXX` is the name of the event to track.

[Back to top](#top)

<a name="custom-contexts" />

### 4.1.1 Custom contexts

In short, custom contexts let you add additional information about the circumstances surrounding an event in the form of an NSDictionary object. Each tracking method accepts an additional optional contexts parameter after all the parameters specific to that method:

```objective-c
- (void) trackPageView:(NSString *)pageUrl
                 title:(NSString *)pageTitle
              referrer:(NSString *)referrer;
- (void) trackPageView:(NSString *)pageUrl
                 title:(NSString *)pageTitle
              referrer:(NSString *)referrer
               context:(NSMutableArray *)context;
- (void) trackPageView:(NSString *)pageUrl
                 title:(NSString *)pageTitle
              referrer:(NSString *)referrer
             timestamp:(double)timestamp;
- (void) trackPageView:(NSString *)pageUrl
                 title:(NSString *)pageTitle
              referrer:(NSString *)referrer
               context:(NSMutableArray *)context
             timestamp:(double)timestamp;
```

The `context` argument should consist of a `NSArray` of `NSDictionary` representing an array of one or more contexts. The format of each individual context element is the same as for an [unstructured event](#unstruct-event).

If a visitor arrives on a page advertising a movie, the context dictionary might look like this:

```json
{
  "schema": "iglu:com.acme_company/movie_poster/jsonschema/2.1.1",
  "data": {
    "movieName": "The Guns of Navarone",
    "posterCountry": "US",
    "posterYear": "1961"
  }
}
```

The corresponding `NSDictionary` would look like this:

```objective-c
NSDictionary *poster = @{
                         @"schema":@"iglu:com.acme_company/movie_poster/jsonschema/1-0-0",
                         @"data": @{
                                 @"movieName": @"The Guns of Navarone",
                                 @"posterCountry": @"US",
                                 @"posterYear": @"1961"
                                 }
                         };
```

Sending the movie poster context with an event looks like this:

```objective-c
[tracker trackStructuredEvent:@"Product"
                       action:@"View"
                        label:nil
                     property:nil
                        value:0
                      context:[NSMutableArray arrayWithArray:@[poster]]];
```

*Note that even if there is only one custom context attached to the event, it still needs to be placed in an array.*

[Back to top](#top)

<a name="tstamp-arg" />

### 4.1.2 Optional timestamp & context argument

In all the trackers, we offer a way to set the timestamp if you want the event to show as tracked at a specific time. If you don't, we create a timestamp while the event is being tracked.

Here is an example:
```objective-c
[tracker trackPageView:@"www.page.com" title:@"Example Page" referrer:@"www.referrer.com"];
[tracker trackPageView:@"www.page.com" title:@"Example Page" referrer:@"www.referrer.com" context:contextArray];
[tracker trackPageView:@"www.page.com" title:@"Example Page" referrer:@"www.referrer.com" timestamp:1234567890];
[tracker trackPageView:@"www.page.com" title:@"Example Page" referrer:@"www.referrer.com" context:contextArray timestamp:1234567890];
```

[Back to top](#top)

<a name="return-value" />

### 4.1.3 Tracker method return values

To be confirmed. As of now, trackers do not return anything.

[Back to top](#top)

<a name="screen-view" />

### 4.2 Track screen views with `trackScreenView:`

Use `trackScreenView:` to track a user viewing a screen (or equivalent) within your app. Arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `name`       | Human-readable name for this screen | No            | NSString*               |
| `id_`        | Unique identifier for this screen   | No            | NSString*               |
| `context`    | Custom context for the event        | No            | NSMutableArray*         |
| `timestamp`  | Optional timestamp for the event    | No            | double                  |

Example:

```objective-c
[t1 trackScreenView:@"HUD > Save Game" screen:@"screen23"];
[t1 trackScreenView:@"HUD > Save Game" screen:nil timestamp:12435678];
[t1 trackScreenView:@"HUD > Save Game" screen:@"screen23" timestamp:12435678];
```

[Back to top](#top)

<a name="page-view" />

### 4.3 Track pageviews with `trackPageView:`

Use `trackPageView:` to track a user viewing a page within your app.

Arguments are:

| **Argument** | **Description**                      | **Required?** | **Validation**          |
|-------------:|:-------------------------------------|:--------------|:------------------------|
| `pageUrl`    | The URL of the page                  | Yes           | NSString*               |
| `pageTitle`  | The title of the page                | Yes           | NSString*               |
| `referrer`   | The address which linked to the page | Yes           | NSString*               |
| `context`    | Custom context for the event         | No            | NSMutableArray*         |
| `timestamp`  | Optional timestamp for the event     | No            | double                  |

Example:

```objective-c
[t1 trackPageView:@"www.example.com" title:@"example" referrer:@"www.referrer.com" context:contextList];
[t1 trackPageView:@"www.example.com" title:@"example" referrer:@"www.referrer.com"];
```

[Back to top](#top)

<a name="ecommerce-transaction" />

### 4.4 Track ecommerce transactions with `trackEcommerceTransaction:`

Use `trackEcommerceTransaction:` to track an ecommerce transaction.
Arguments:

| **Argument**  | **Description**                      | **Required?** | **Validation**  |
|--------------:|:-------------------------------------|:--------------|:----------------|
| `orderId`     | ID of the eCommerce transaction      | Yes           | NSString*       |
| `totalValue`  | Total transaction value              | Yes           | float           |
| `affiliation` | Transaction affiliation              | No            | NSString*       |
| `taxValue`    | Transaction tax value                | No            | float           |
| `shipping`    | Delivery cost charged                | No            | float           |
| `city`        | Delivery address city                | No            | NSString*       |
| `state`       | Delivery address state               | No            | NSString*       |
| `country`     | Delivery address country             | No            | NSString*       |
| `currency`    | Transaction currency                 | No            | NSString*       |
| `items`       | Items in the transaction             | Yes           | NSMutableArray* |
| `context`     | Custom context for the event         | No            | NSMutableArray* |
| `tstamp`      | When the transaction event occurred  | No            | double          |

`trackEcommerceTransaction:` fires multiple events: one "transaction" event for the transaction as a whole, and one "transaction item" event for each element of the `items` array. Each transaction item event will have the same timestamp, orderId, and currency as the main transaction event.

The `items` argument is an `NSMutableArray` containing an `NSDictionary` for each item in the transaction. There is a convenience constructor for each item called `trackEcommerceTransactionItem:`. Arguments:

| **Field**       | **Description**                     | **Required?** | **Validation**  |
|----------------:|:------------------------------------|:--------------|:----------------|
| `sku`           | Item SKU                            | Yes           | NSString*       |
| `price`         | Item price                          | Yes           | float           |
| `quantity`      | Item quantity                       | Yes           | float           |
| `name`          | Item name                           | No            | NSString*       |
| `category`      | Item category                       | No            | NSString*       |
| `context`       | Custom context for the event        | No            | NSMutableArray* |
| `currency`      | Transaction currency                | No            | NSString*       |
| `context`       | Custom context for the event        | No            | NSMutableArray* |
| `tstamp`        | When the transaction event occurred | No            | double          |

Example of tracking a transaction containing one item:

```objective-c
NSString *transactionID = @"6a8078be";
NSMutableArray *itemArray = [NSMutableArray array];

[itemArray addObject:[t trackEcommerceTransactionItem:transactionID
                                                  sku:@"pbz0026"
                                                 name:@"Hot Chocolate"
                                             category:@"Drink"
                                                price:0.75F
                                             quantity:1
                                             currency:@"USD"]];
[t trackEcommerceTransaction:transactionID
                  totalValue:350
                 affiliation:@"no_affiliate"
                    taxValue:10
                    shipping:15
                        city:@"Boston"
                       state:@"Massachusetts"
                     country:@"USA"
                    currency:@"USD"
                       items:itemArray];
```

[Back to top](#top)

<a name="struct-event" />

### 4.5 Track structured events with `trackStructuredEvent:`

Use `trackStructuredEvent:` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument** | **Description**                                                  | **Required?** | **Validation**   |
|-------------:|:-----------------------------------------------------------------|:--------------|:-----------------|
| `category`   | The grouping of structured events which this `action` belongs to | Yes           | NSString*        |
| `action`     | Defines the type of user interaction which this event involves   | Yes           | NSString*        |
| `label`      | A string to provide additional dimensions to the event data      | Yes           | NSString*        |
| `property`   | A string describing the object or the action performed on it     | Yes           | NSString*        |
| `value`      | A value to provide numerical data about the event                | Yes           | int              |
| `context`    | Custom context for the event                                     | No            | NSMutableArray*  |
| `timestamp`  | Optional timestamp for the event                                 | No            | double           |

Example:

```objective-c
[t1 trackStructuredEvent:@"shop" action:@"add-to-basket" label:@"Add To Basket" property:@"pcs" value:27];
[t1 trackStructuredEvent:@"shop" action:@"add-to-basket" label:@"Add To Basket" property:@"pcs" value:27 timestamp:1234569];
```

[Back to top](#top)

<a name="unstruct-event" />

### 4.6 Track unstructured events with `trackUnstructuredEvent:`

Custom unstructured events are a flexible tool that enable Snowplow users to define their own event types and send them into Snowplow.

When a user sends in a custom unstructured event, they do so as a JSON of name-value properties, that conforms to a JSON schema defined for the event earlier.

Use `trackUnstructuredEvent:` to track a custom event which consists of a name and an unstructured set of properties. This is useful when:

* You want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow), or
* You want to track events which have unpredictable or frequently changing properties

The arguments are as follows:

| **Argument**   | **Description**                   |  **Required?** |    **Validation**   |
|---------------:|:----------------------------------|:---------------|:--------------------|
| `eventJson`    | The properties of the event       | Yes            | NSDictionary*       |
| `context`      | Custom context for the event      | No             | NSMutableArray*     |
| `timestamp`    | Optional timestamp for the event  | No             | double              |

Example:

```objective-c
- (void) trackUnstructuredEvent:(NSDictionary *)eventJson
                        context:(NSMutableArray *)context
                      timestamp:(double)timestamp;
```

If you supply a `NSDictionary*`, make sure that this top-level contains your `schema` and `data` keys, and then store your `data` properties as a child `NSDictionary*`.

Example:

```objective-c
NSDictionary *event = @{
                          @"schema":@"iglu:com.acme/save_game/jsonschema/1-0-0",
                          @"data": @{
                                  @"level": @23,
                                  @"score": @56473
                                  }
                          };
tracker trackUnstructuredEvent:event
                        context:nil
                      timestamp:12345678;
```

For more on JSON schema, see the [blog post][self-describing-jsons].

[Back to top](#top)

<a name="timing" />

### 4.7 Track user timings with `trackTimingWithCategory:`

Use `trackTiming:` to track a user timing in your app - for example, how long a game took to load, or how long an in-app purchase took to download. The fields are as follows:

| **Argument** | **Description**                                                  | **Required?** | **Validation**   |
|-------------:|:-----------------------------------------------------------------|:--------------|:-----------------|
| `category`   | Categorizing timing variables into logical groups (e.g API calls, asset loading) | Yes | NSString*  |
| `variable`   | Identify the timing being recorded                               | Yes           | NSString*        |
| `timing`     | The number of milliseconds in elapsed time to report             | Yes           | NSUInteger       |
| `label`      | Optional description of this timing                              | Yes           | NSString*        |
| `context`    | Custom context for the event                                     | No            | NSMutableArray*  |
| `timestamp`  | Optional timestamp for the event                                 | No            | double           |

Example:

```objective-c
[t1 trackTimingWithCategory:@"Application"
                   variable:@"Background"
                     timing:324
                      label:@"5231804123"];

[t1 trackTimingWithCategory:@"Application"
                   variable:@"Background"
                     timing:324
                      label:@"5231804123"
                  timestamp:1234569];
```

[Back to top](#top)

<a name="emitters" />

## 5. Sending events: `SnowplowEmitter`

Events created by the Tracker are sent to a collector using a `SnowplowEmitter` instance. You can create one using one of the init methods:

```objective-c
- (id) initWithURLRequest:(NSURL *)url
               httpMethod:(NSString *)method
             bufferOption:(enum SnowplowBufferOptions)option
          emitterCallback:(id<RequestCallback>)callback;

- (id) initWithURLRequest:(NSURL *)url
               httpMethod:(NSString *)method
             bufferOption:(enum SnowplowBufferOptions)option;

- (id) initWithURLRequest:(NSURL *)url
               httpMethod:(NSString* )method;
```

For example:

```objective-c
NSURL *url = [[NSURL alloc] initWithString:@"https://collector.acme.net"];
SnowplowEmitter emitter = [[SnowplowEmitter alloc] initWithURLRequest:url
                                                           httpMethod:@"POST"
                                                         bufferOption:SnowplowBufferInstant];
SnowplowEmitter emitter2 = [[SnowplowEmitter alloc] initWithURLRequest:url
                                                            httpMethod:@"GET"];
```

<a name="buffer" />

### 5.1 Using a buffer

A buffer is used to group events together in bulk before sending them. This is especially handy to reduce network usage. By default, the SnowplowEmitter buffers up to 10 events before sending them.

You can set this during the creation of a `SnowplowEmitter` object or using the setter `-(void)setBufferOption:`

```objective-c
NSURL *url = [[NSURL alloc] initWithString:@"https://collector.acme.net"];
SnowplowEmitter emitter = [[SnowplowEmitter alloc] initWithURLRequest:url
                                                           httpMethod:@"POST"
                                                         bufferOption:SnowplowBufferInstant];
SnowplowEmitter emitter2 = [[SnowplowEmitter alloc] initWithURLRequest:url
                                                           httpMethod:@"POST"
                                                         bufferOption:SnowplowBufferDefault];
[emitter setBufferOption:SnowplowBufferInstant];
```

Here are all the posibile options that you can use:
|         **Option**         | **Description**                                    |
|---------------------------:|:---------------------------------------------------|
| `SnowplowBufferInstant`    | Events are sent as soon as they are created        |
| `SnowplowBufferDefault`    | Sends events in a group when 10 events are created |

<a name="http-method" />

### 5.2 Choosing the HTTP method

Snowplow supports receiving events via GET requests, but will soon have POST support. In a GET request, each event is sent in individual request. With POST requests, events are bundled together in one request.

Here are all the posibile options that you can use:

| **Option**  | **Description**                                                            |
|------------:|:---------------------------------------------------------------------------|
| `@"GET"`    | Events are sent individually as GET requests                               |
| `@"POST"`   | Events are sent in a group when 10 events are received in one POST request |

<a name="emitter-callback" />

### 5.3 Adding an Emitter Callback

You are now also able to include an emitter callback which will return the count of successful and failed events.

To implement you will need to:

* Add the `RequestCallback` protocol to your header file:

```objective-c
// Example from the SnowplowDemo -> ViewController.h file:
@interface ViewController : UIViewController <UITextFieldDelegate, RequestCallback>

// Extra Example
@interface MyObjcClass : NSObject <RequestCallback>
```

* In your paired `.m` file add the following functions:

```objective-c
// Define Callback Functions
- (void) onSuccess:(NSInteger)successCount {
    // Do something with result
}

- (void) onFailure:(NSInteger)successCount failure:(NSInteger)failureCount {
    // Do something with results
}
```

* Construct the SnowplowEmitter like so:

```objective-c
NSURL *url = [[NSURL alloc] initWithString:@"https://collector.acme.net"];

SnowplowEmitter emitter = [[SnowplowEmitter alloc] initWithURLRequest:url
                                                           httpMethod:@"POST"
                                                         bufferOption:SnowplowBufferInstant
                                                      emitterCallback:self];
```

The `self` will work only if you have declared the callback functions in the same class as you are creating the Emitter from.

<a name="http-request" />

### 5.4 Sending HTTP requests

You can set this during the creation of a `SnowplowEmitter` object:
```objective-c
NSURL *url = [[NSURL alloc] initWithString:@"https://collector.acme.net"];
SnowplowEmitter emitter = [[SnowplowEmitter alloc] initWithURLRequest:url
                                                           httpMethod:@"POST"
                                                         bufferOption:SnowplowBufferInstant];
SnowplowEmitter emitter2 = [[SnowplowEmitter alloc] initWithURLRequest:url
                                                           httpMethod:@"GET"
                                                         bufferOption:SnowplowBufferDefault];
```

[Back to top](#top)

[ios-0.1]: https://github.com/snowplow/snowplow/wiki/iOS-Tracker-v0.1
[ios-0.2]: https://github.com/snowplow/snowplow/wiki/iOS-Tracker-v0.2
[ios-0.3]: https://github.com/snowplow/snowplow/wiki/iOS-Tracker-v0.3

[base64]: https://en.wikipedia.org/wiki/Base64
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
