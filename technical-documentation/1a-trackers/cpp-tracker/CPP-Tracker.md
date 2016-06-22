<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow technical documentation) > [**Trackers**](trackers) > CPP Tracker

## Contents

- 1. [Overview](#overview)  
- 2. [Initialization](#init)  
  - 2.1 [Import the library](#importing)
  - 2.2 [Creating a tracker](#create-tracker)
    - 2.2.1 [`emitter`](#emitter)  
    - 2.2.2 [`subject`](#subject)
    - 2.2.3 [`client_session`](#client-session)
    - 2.2.4 [`platform`](#platform)
    - 2.2.5 [`app_id`](#app-id)
    - 2.2.6 [`name_space`](#name-space)
    - 2.2.7 [`use_base64`](#base64)
    - 2.2.8 [`desktop_context`](#desktop-context)
- 3. [Adding extra data: the Subject class](#subject-class)
  - 3.1 [`set_user_id`](#set-user-id)
  - 3.2 [`set_screen_resolution`](#set-screen-resolution)
  - 3.3 [`set_viewport`](#set-viewport-dimensions)
  - 3.4 [`set_color_depth`](#set-color-depth)
  - 3.5 [`set_timezone`](#set-timezone)
  - 3.6 [`set_language`](#set-lang)
- 4. [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Custom contexts](#custom-contexts)
    - 4.1.2 [Optional timestamp argument](#tstamp-arg)
    - 4.1.3 [Optional true-timestamp argument](#true-tstamp-arg)
    - 4.1.4 [Optional event-id argument](#event-id-arg)
  - 4.2 [`track_self_describing_event()`](#unstruct-event)
  - 4.3 [`track_screen_view()`](#screen-view)
  - 4.4 [`track_struct_event()`](#struct-event)
  - 4.5 [`track_timing()`](#timing-event)
- 5. [Emitters](#emitter)
  - 5.1 [Under the hood](#emitter-inner-workings)
- 6. [Client Sessions](#sessions)

<a name="overview" />
## 1. Overview

The [Snowplow C++ Tracker](https://github.com/snowplow/snowplow-cpp-tracker) allows you to track Snowplow events from your C++ apps, games and servers.

There are three basic types of object you will create when using the Snowplow C++ Tracker: subjects, emitters, and trackers.

A subject represents a user whose events are tracked. A tracker constructs events and sends them to an emitter. The emitter then sends the event to the endpoint you configure; a valid Snowplow Collector.

<a name="init" />
## 2 Initialization

Assuming you have completed the [[CPP Tracker Setup]] for your project, you are now ready to initialize the C++ Tracker.

<a name="importing" />
### 2.1 Import the library

Import the C++ Tracker library like so:

```c++
#include "tracker.hpp"
```

That's it - you are now ready to initialize a tracker instance.

<a name="create-tracker" />
### 2.2 Creating a tracker

You will first need to create an Emitter to which events will be sent:

```c++
#include "tracker.hpp"

Emitter emitter("com.acme.collector", Emitter::Method::POST, Emitter::Protocol::HTTP, 500, 52000, 52000, "events.db");
Tracker *t = Tracker::init(emitter, NULL, NULL, NULL, NULL, NULL, NULL);
```

The Tracker can now be used.  It is created as a static singleton object so as to avoid conflicts between multiple Trackers reading and writing to a single database. You can access a pointer to the Tracker by either the `Tracker::init(...)` function or by the `Tracker::instance()` function.  To ensure the Tracker is cleaned up properly you will need to run `Tracker::close()` in your main destructor.

__NOTE__: If you attempt to access the `instance` without having first run `init` an exception will be thrown.

The optional tracker arguments:

| **Argument Name**    | **Description**                               | **Required?** | **Default**  |
|---------------------:|:----------------------------------------------|:--------------|:-------------|
| `emitter`            | The emitter to which events are sent          | Yes           | `NULL`       |
| `subject`            | The user being tracked                        | No            | `NULL`       |
| `client_session`     | Client session recording                      | No            | `NULL`       |
| `platform`           | The platform the Tracker is running on        | No            | `srv`        |
| `app_id`             | The application ID                            | No            | ``           |
| `name_space`         | The name of the tracker instance              | No            | ``           |
| `use_base64`         | Whether to enable [base 64 encoding] [base64] | No            | `true`       |
| `desktop_context`    | Whether to add a `desktop_context` to events  | No            | `true`       |

A more complete example:

```c++
Emitter emitter("com.acme.collector", Emitter::Method::GET, Emitter::Protocol::HTTPS, 500, 52000, 52000, "sp.db");

Subject subject;
subject.set_user_id("a-user-id");
subject.set_screen_resolution(1920, 1080);
subject.set_viewport(1080, 1080);
subject.set_color_depth(32);
subject.set_timezone("GMT");
subject.set_language("EN");

ClientSession client_session("sp.db", 5000, 5000, 2500);

string platform = "pc";
string app_id = "openage";
string name_space = "sp-pc";
bool base64 = false;
bool desktop_context = true;

Tracker *t = Tracker::init(emitter, &subject, &client_session, &platform, &app_id, &name_space, &base64, &desktop_context);
```

__NOTE__: To ensure all of your events are sent before closing your application you can call the tracker flush function which is a blocking call that will send everything in the database and then will join the daemon thread to the calling thread.

<a name="emitter" />
#### 2.2.1 `emitter`

Accepts an argument of an Emitter instance pointer; if the object is `NULL` will throw an exception. See [Emitters](#emitters) for more on emitter configuration.

<a name="subject" />
#### 2.2.2 `subject`

The user which the Tracker will track. Accepts an argument of a [Subject](#subject-class) instance pointer. 

You don't need to set this during Tracker construction; you can use the `tracker.set_subject(...)` method afterwards. In fact, you don't need to create a subject at all. If you don't, though, your events won't contain user-specific data such as timezone and language.

<a name="client-session" />
#### 2.2.3 `client_session`

The client sessions which the Tracker will attach to each event. Accepts an argument of a [ClientSession](#sessions) instance pointer. 

Adds the ability to attach a ClientSession context to each event that leaves the Tracker.  This object operates on a daemon thread and will persistently store information about the sessions that have occurred for the life of the application - unless the database is destroyed.

<a name="platform" />
#### 2.2.4 `platform`

By default we assume the Tracker will be running in a server environment.  To override this provide your own platform string.

<a name="app-id" />
#### 2.2.5 `app_id`

The `app_id` argument lets you set the application ID to any string.

<a name="name-space" />
#### 2.2.6 `name_space`

If provided, the `name_space` argument will be attached to every event fired by the new tracker. This allows you to later identify which tracker fired which event if you have multiple trackers running.

<a name="base64" />
#### 2.2.7 `use_base64`

By default, unstructured events and custom contexts are encoded using Base64 to ensure that no data is lost or corrupted.

<a name="desktop-context" />
#### 2.2.8 `desktop_context`

The `desktop_context` gathers extra information about the device it is running on and sends it along with every event that is made by the Tracker.

An example of the data in this context:

```json
{
	"deviceManufacturer": "Apple Inc.",
	"deviceModel": "MacPro3,1",
	"deviceProcessorCount": 8,
	"osIs64Bit": true,
	"osServicePack": "",
	"osType": "macOS",
	"osVersion": "10.11.2"
}
```

For more information the raw JsonSchema can be found [here](https://raw.githubusercontent.com/snowplow/iglu-central/master/schemas/com.snowplowanalytics.snowplow/desktop_context/jsonschema/1-0-0).

[Back to top](#top)

<a name="subject-class" />
## 3. Adding extra data: The Subject class

You may have additional information about your application's environment, current user and so on, which you want to send to Snowplow with each event.

Create a subject like this:

```c++
Subject subject;
```

The Subject class has a set of `set...()` methods to attach extra data relating to the user to all tracked events:

* [`set_user_id`](#set-user-id)
* [`set_screen_resolution`](#set-screen-resolution)
* [`set_viewport`](#set-viewport-dimensions)
* [`set_color_depth`](#set-color-depth)
* [`set_timezone`](#set-timezone)
* [`set_language`](#set-lang)

We will discuss each of these in turn below:

<a name="set-user-id" />
### 3.1 Set user ID with `set_user_id`

You can set the user ID to any string:

```c++
s.set_user_id( "{{USER ID}}" );
```

Example:

```c++
s.set_user_id("alexd");
```

[Back to top](#top)

<a name="set-screen-resolution" />
### 3.2 Set screen resolution with `set_screen_resolution`

If your code has access to the device's screen resolution, then you can pass this in to Snowplow too:

```c++
s.set_screen_resolution( {{WIDTH}}, {{HEIGHT}} );
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```c++
s.set_screen_resolution(1366, 768);
```

[Back to top](#top)

<a name="set-viewport-dimensions" />
### 3.3 Set viewport dimensions with `set_viewport`

If your code has access to the viewport dimensions, then you can pass this in to Snowplow too:

```c++
s.set_viewport( {{WIDTH}}, {{HEIGHT}} );
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```c++
s.set_viewport(300, 200);
```

[Back to top](#top)

<a name="set-color-depth" />
### 3.4 Set color depth with `set_color_depth`

If your code has access to the bit depth of the device's color palette for displaying images, then you can pass this in to Snowplow too:

```c++
s.set_color_depth( {{BITS PER PIXEL}} );
```

The number should be a positive integer, in bits per pixel. Example:

```c++
s.set_color_depth(32);
```

[Back to top](#top)

<a name="set-timezone" />
### 3.5 Set timezone with `set_timezone`

This method lets you pass a user's timezone in to Snowplow:

```c++
s.set_timezone( {{TIMEZONE}} );
```

The timezone should be a string:

```c++
s.set_timezone("Europe/London");
```

[Back to top](#top)

<a name="set-lang" />
### 3.6 Set the language with `set_language`

This method lets you pass a user's language in to Snowplow:

```c++
s.set_language( {{LANGUAGE}} );
```

The language should be a string:

```c++
s.set_language('en');
```

[Back to top](#top)

<a name="events" />
## 4. Tracking specific events

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your websites and apps. We are constantly growing the range of functions available in order to capture that data more richly.

Tracking methods supported by the C++ Tracker at a glance:

| **Function**                                       | **Description**                                        |
|---------------------------------------------------:|:-------------------------------------------------------|
| [`track_struct_event()`](#struct-event)            | Track a Snowplow custom structured event               |
| [`track_self_describing_event()`](#unstruct-event) | Track a Snowplow custom unstructured event             |
| [`track_screen_view()`](#screen-view)              | Track the user viewing a screen within the application |
| [`track_timing()`](#timing-event)                  | Track a timing event                                   |

<a name="common" />
### 4.1 Common

All events are tracked with specific methods on the tracker instance, of the form `track_xxx()`, where `xxx` is the name of the event to track.

<a name="custom-contexts" />
### 4.1.1 Custom contexts

In short, custom contexts let you add additional information about the circumstances surrounding an event in the form of a vector of SelfDescribingJson objects. Each tracking method accepts an additional optional contexts parameter, which should be a vector of contexts:

```c++
vector<SelfDescribingJson> contexts;
```

If a visitor arrives on a page advertising a movie, the dictionary for a single custom context in the list might look like this:


```c++
vector<SelfDescribingJson> contexts;
SelfDescribingJson sdj(
  "iglu:com.acme_company/movie_poster/jsonschema/2.1.1",
  "{\"movie_name\":\"Solaris\",\"poster_country\":\"JP\"}"_json
);
contexts.push_back(sdj);
```

This is how to fire a structured event with the above custom context:

```c++
Tracker::StructuredEvent se("category", "action");
se.contexts = contexts;

Tracker::instance()->track_struct_event(se);
```

Note that even though there is only one custom context attached to the event, it still needs to be placed in a list.

__INFO__: We use the excellent json library from Github community member [Niels (nlohmann)](https://github.com/nlohmann) for all JSON parsing.  For more information on using this library please visit the [Technical information on Github](https://github.com/nlohmann/json).

<a name="tstamp-arg" />
### 4.1.2 Optional timestamp argument

Each `track...()` method supports an optional timestamp argument; this allows you to manually override the timestamp attached to this event. The timestamp should be in milliseconds since the Unix epoch.

If you do not pass this timestamp in as an argument, then the C++ Tracker will use the current time to be the timestamp for the event.

Here is an example tracking a structured event and supplying the optional timestamp argument:

```c++
Tracker::StructuredEvent se("category", "action");
se.timestamp(1368725287000);

Tracker::instance()->track_struct_event(se);
```

<a name="true-tstamp-arg" />
### 4.1.3 Optional true-timestamp argument

Each `track...()` method supports an optional true-timestamp argument; this allows you to provide the true-timestamp attached to this event to help with the timing of events in multiple timezones. The timestamp should be in milliseconds since the Unix epoch.

Here is an example tracking a structured event and supplying the optional true-timestamp argument:

```c++
Tracker::StructuredEvent se("category", "action");

// As it is optional you will need to pass the address for this value
unsigned long long true_tstamp = "1368725287000";
se.true_timestamp = &true_tstamp;

Tracker::instance()->track_struct_event(se);
```

<a name="event-id-arg" />
### 4.1.4 Optional event ID argument

Each `track...()` method supports an optional event id argument; this allows you to manually override the event ID attached to this event. The event ID should be a valid version 4 UUID string.

Here is an example tracking a structured event and supplying the optional event ID argument:

```c++
StructuredEvent se("category", "action");
se.event_id = "ba55081a-a58c-40b4-afb2-4915dd149200";

Tracker::instance()->track_struct_event(se);
```

[Back to top](#top)

<a name="unstruct-event" />
### 4.2 Track SelfDescribing/Unstructured events with `track_self_describing_event()`

Use `track_self_describing_event()` to track a custom event which consists of a name and an unstructured set of properties. This is useful when:

* You want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow), or
* You want to track events which have unpredictable or frequently changing properties

The arguments are as follows:

| **Argument**     | **Description**                      | **Required?** | **Validation**             |
|-----------------:|:-------------------------------------|:--------------|:---------------------------|
| `event`          | The properties of the event          | Yes           | SelfDescribingJson         |
| `timestamp`      | When the event occurred              | No            | unsigned long long         |
| `event_id`       | The event ID                         | No            | string                     |
| `true_timestamp` | The true time of event               | No            | *unsigned long long        |
| `contexts`       | Custom contexts for the event        | No            | vector<SelfDescribingJson> |

Example:

```c++
// Create a JSON containing your data
json data = "{\"level\":5,\"saveId\":\"ju302\",\"hardMode\":true}"_json;

// Create a new SelfDescribingJson
SelfDescribingJson sdj("iglu:com.example_company/save-game/jsonschema/1-0-2", data);

Tracker::SelfDescribingEvent sde(sdj);
Tracker::instance()->track_self_describing_event(sde);
```

For more on JSON schema, see the [blog post] [self-describing-jsons].

[Back to top](#top)

<a name="screen-view" />
### 4.3 Track screen views with `track_screen_view()`

Use `track_screen_view()` to track a user viewing a screen (or equivalent) within your app:

| **Argument**     | **Description**                      | **Required?** | **Type**                   |
|-----------------:|:-------------------------------------|:--------------|:---------------------------|
| `name`           | Human-readable name for this screen  | No            | *string                    |
| `id`             | Unique identifier for this screen    | No            | *string                    |
| `timestamp`      | When the event occurred              | No            | unsigned long long         |
| `event_id`       | The event ID                         | No            | string                     |
| `true_timestamp` | The true time of event               | No            | *unsigned long long        |
| `contexts`       | Custom contexts for the event        | No            | vector<SelfDescribingJson> |

Although name and id are not individually required, at least one must be provided or the event will fail validation and subsequently throw an exception.

Example:

```c++
string name = "Screen ID - 5asd56";

Tracker::ScreenViewEvent sve;
sve.name = &name;

Tracker::instance()->track_screen_view(sve);
```

[Back to top](#top)

<a name="struct-event" />
### 4.4 Track structured events with `TrackStructEvent()`

Use `TrackStructEvent()` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument**     | **Description**                                                  | **Required?** | **Validation**             |
|-----------------:|:-----------------------------------------------------------------|:--------------|:---------------------------|
| `category`       | The grouping of structured events which this `action` belongs to | Yes           | string                     |
| `action`         | Defines the type of user interaction which this event involves   | Yes           | string                     |
| `label`          | A string to provide additional dimensions to the event data      | No            | *string                    |
| `property`       | A string describing the object or the action performed on it     | No            | *string                    |
| `value`          | A value to provide numerical data about the event                | No            | *double                    |
| `timestamp`      | When the event occurred                                          | No            | unsigned long long         |
| `event_id`       | The event ID                                                     | No            | string                     |
| `true_timestamp` | The true time of event                                           | No            | *unsigned long long        |
| `contexts`       | Custom contexts for the event                                    | No            | vector<SelfDescribingJson> |

Example:

```c++
Tracker::StructuredEvent se("shop", "add-to-basket");
se.property = "pcs";
se.value = 25.6;

Tracker::instance()->track_struct_event(se);
```

[Back to top](#top)

<a name="timing-event" />
### 4.5 Track timing events with `TrackTiming()`

Use `TrackTiming()` to track a timing event.

The arguments are as follows:

| **Argument**     | **Description**                      | **Required?** | **Validation**             |
|-----------------:|:-------------------------------------|:--------------|:---------------------------|
| `category`       | The category of the event            | Yes           | *string                    |
| `variable`       | The variable of the event            | Yes           | *string                    |
| `timing`         | The timing of the event              | Yes           | *int64                     |
| `label`          | The label of the event               | No            | *string                    |
| `timestamp`      | When the event occurred              | No            | unsigned long long         |
| `event_id`       | The event ID                         | No            | string                     |
| `true_timestamp` | The true time of event               | No            | *unsigned long long        |
| `contexts`       | Custom contexts for the event        | No            | vector<SelfDescribingJson> |

Example:

```c++
Tracker::TimingEvent te("category", "variable", 123);
Tracker::instance()->track_timing(te);
```

[Back to top](#top)

<a name="emitter" />
## 5. Emitters

The Tracker instance must be initialized with an emitter. This section will go into more depth about the Emitter and how it works under the hood.

```c++
Emitter emitter("com.acme", Emitter::Method::POST, Emitter::Protocol::HTTP, 52000, 52000, 500, "sp.db");
```

There are other optional arguments:

| **Argument Name**     | **Description**                                | **Required?** |
|----------------------:|:-----------------------------------------------|:--------------|
| `uri`                 | The URI to send events to                      | Yes           |
| `method`              | The request type to use (GET or POST)          | Yes           |
| `protocol`            | The protocol to use (http or https)            | Yes           |
| `send_limit`          | The maximum amount of events to send at a time | Yes           |
| `byte_limit_post`     | The byte limit when sending a POST request     | Yes           |
| `byte_limit_get`      | The byte limit when sending a GET request      | Yes           |
| `db_name`             | Defines the path and file name of the database | Yes           |

The `db_name` can be any valid path on your host file system (that can be created with the current user).  By default it will create the required files wherever the application is being run from.

[Back to top](#top)

<a name="emitter-inner-workings">
### 5.1 Under the hood

Once the emitter receives an event from the Tracker a few things start to happen:

* The event is added to a local Sqlite3 database (blocking execution)
* A long running daemon thread is started which will continue to send events as long as they can be found in the database (asynchronous)
* The emitter loop will grab a range of events from the database up until the `SendLimit` as noted __above_
* The emitter will send all of these events as determined by the Request, Protocol and ByteLimits
  - Each request is sent in its thread.
* Once sent it will process the results of all the requests sent and will remove all successfully sent events from the database

[Back to top](#top)

<a name="sessions" />
## 6. Client Sessions

You can optionally decide to include Client Sessionization.  This object will keep track of your users sessions and can be configured to timeout after a certain amount of inactivity.  Activity is determined by how often events are sent with the Tracker - so you will need to send events to keep the current session active.

```c++
ClientSession client_session("sp.db", 5000, 5000, 2500);
```

__NOTE__: The database path used here _must_ be the same as the one used in the Emitter.

| **Argument Name**     | **Description**                                | **Required?** |
|----------------------:|:-----------------------------------------------|:--------------|
| `db_name`             | Defines the path and file name of the database | Yes           |
| `foreground_timeout`  | The timeout when the app is in focus           | Yes           |
| `background_timeout`  | The timeout when the app is backgrounded       | Yes           |
| `check_interval`      | The time between checks                        | Yes           |

All timing metrics are configured in milliseconds.

To set the background/foreground state you will need to detect this and then set this on the ClientSession object like so:

```c++
client_session.set_is_background(true || false);
```

[jsonschema]: http://snowplowanalytics.com/blog/2014/05/13/introducing-schemaver-for-semantic-versioning-of-schemas/
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
[base64]: https://en.wikipedia.org/wiki/Base64
