<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Trackers**](trackers) > Python Tracker

*This page refers to version 0.5.0 of the Snowplow Python Tracker. Documentation for other versions is available:*

*[Version 0.2][python-0.2]*

*[Version 0.3][python-0.3]*

*[Version 0.4][python-0.4]*

*[Version 0.6][python-0.6]*

*[Version 0.7][python-0.7]*

*[Version 0.8][python-latest]*

## Contents

- 1. [Overview](#overview)
- 2. [Initialization](#init)
  - 2.1 [Importing the module](#importing)
  - 2.2 [Creating a tracker](#create-tracker)
    - 2.2.1 [`emitters`](#emitter)
    - 2.2.2 [`subject`](#subject)
    - 2.2.3 [`namespace`](#namespace)
    - 2.2.4 [`app_id`](#app-id)
    - 2.2.5 [`encode_base64`](#base64)
- 3. [Adding extra data: the Subject class](#subject-class)
  - 3.1 [`set_platform`](#set-platform)
  - 3.2 [`set_user_id`](#set-user-id)
  - 3.3 [`set_screen_resolution`](#set-screen-resolution)
  - 3.4 [`set_viewport`](#set-viewport-dimensions)
  - 3.5 [`set_color_depth`](#set-color-depth)
  - 3.6 [`set_timezone`](#set-timezone)
  - 3.7 [`set_lang`](#set-lang)
- 4. [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Custom contexts](#custom-contexts)
    - 4.1.2 [Optional timestamp argument](#tstamp-arg)
    - 4.1.3 [Tracker method return values](#return-values)
  - 4.2 [`track_screen_view()`](#screen-view)
  - 4.3 [`track_page_view()`](#page-view)
  - 4.4 [`track_ecommerce_transaction()`](#ecommerce-transaction)
  - 4.5 [`track_ecommerce_transaction_item()`](#ecommerce-transaction-item)
  - 4.6 [`track_struct_event()`](#struct-event)
  - 4.7 [`track_unstruct_event()`](#unstruct-event)
- 5. [Emitters](#emitter)
  - 5.1 [The basic Emitter class](#base-emitter)
  - 5.2 [The AsyncEmitter class](#async-emitter)
  - 5.3 [The CeleryEmitter class](#celery-emitter)
  - 5.4 [The RedisEmitter class](#redis-emitter)
  - 5.5 [Manual flushing](#manual-flushing)
  - 5.6 [Multiple emitters](#multiple-emitters)
  - 5.7 [Custom emitters](#custom-emitters)
- 6 [Contracts](#contracts)
- 7 [Logging](#logging)
- 8 [The RedisWorker class](#redis-worker)


<a name="overview" />

## 1. Overview

The [Snowplow Python Tracker](https://github.com/snowplow/snowplow-python-tracker) allows you to track Snowplow events from your Python apps and games.

The tracker should be straightforward to use if you are comfortable with Python development; any prior experience with Snowplow's [[JavaScript Tracker]] or [[Lua Tracker]], Google Analytics or Mixpanel (which have similar APIs to Snowplow) is helpful but not necessary.

Note that this tracker has access to a more restricted set of Snowplow events than the [[JavaScript Tracker]] and covers almost all the events from the [[Lua Tracker]].

There are three basic types of object you will create when using the Snowplow Python Tracker: subjects, emitters, and trackers.

A subject represents a user whose events are tracked. A tracker constructs events and sends them to one or more emitters. Each emitter then sends the event to the endpoint you configure. This will usually be a Snowplow collector, but could also be a Redis database or Celery task queue.

<a name="init" />

## 2 Initialization

Assuming you have completed the [[Python Tracker Setup]] for your Python project, you are now ready to initialize the Python Tracker.

<a name="importing" />

### 2.1 Importing the module

Require the Python Tracker's module into your Python code like so:

```python
from snowplow_tracker import Subject, Tracker, Emitter
```

That's it - you are now ready to initialize a tracker instance.

[Back to top](#top)

<a name="create-tracker" />

### 2.2 Creating a tracker

The simplest tracker initialization only requires you to provide the URI of the collector to which the tracker will log events:

```python
e = Emitter("d3rkrsqld9gmqf.cloudfront.net")
t = Tracker(e)
```

There are other optional keyword arguments:

| **Argument Name** | **Description**                      | **Required?** | **Default**          |
|-------------:|:-------------------------------------|:--------------|:------------------------|
| `emitters`      | The emitter to which events are sent       | Yes           | `None`        |
| `subject`   | The user being tracked               |         No            | `subject.Subject()` |
| `namespace`  | The name of the tracker instance     |  No           |  `None` |
| `app_id` | The application ID          | No           | `None`         |
| `encode_base64` | Whether to enable [base 64 encoding][base64] | No | `True`  |

A more complete example:

```python
tracker = Tracker( Emitter("d3rkrsqld9gmqf.cloudfront.net") , namespace="cf", app_id="cf29ea", encode_base64=False)
```

[Back to top](#top)

<a name="emitter" />

#### 2.2.1 `emitters`

This can be a single emitter or an array containing at least one emitter. The tracker will send events to these emitters, which will in turn send them to a collector. See [Emitters](#emitters) for more on emitter configuration.

<a name="subject" />

#### 2.2.2 `subject`

The user which the Tracker will track. This should be an instance of the [Subject](#subject) class. You don't need to set this during Tracker construction; you can use the `Tracker.set_subject` method afterwards. In fact, you don't need to create a subject at all. If you don't, though, your events won't contain user-specific data such as timezone and language.

<a name="namespace" />

#### 2.2.3 `namespace`

If provided, the `namespace` argument will be attached to every event fired by the new tracker. This allows you to later identify which tracker fired which event if you have multiple trackers running.

<a name="app-id" />

#### 2.2.4 `app_id`

The `app_id` argument lets you set the application ID to any string.

<a name="base64" />

#### 2.2.5 `encode_base64`

By default, unstructured events and custom contexts are encoded into Base64 to ensure that no data is lost or corrupted. You can turn encoding on or off using the Boolean `encode_base64` argument.

[Back to top](#top)

<a name="subject-class" />

## 3. Adding extra data: The Subject class

You may have additional information about your application's environment, current user and so on, which you want to send to Snowplow with each event.

Create a subject like this:

```python
from snowplow_tracker import Subject
s = subject()
```

The Subject class has a set of `set...()` methods to attach extra data relating to the user to all tracked events:

* [`set_platform`](#set-platform)
* [`set_user_id`](#set-user-id)
* [`set_screen_resolution`](#set-screen-resolution)
* [`set_viewport`](#set-viewport-dimensions)
* [`set_color_depth`](#set-color-depth)
* [`set_timezone`](#set-timezone)
* [`set_lang`](#set-lang)

If you initialize a `Tracker` instance without a subject, a default `Subject` instance will be attached to the tracker. You can access that subject like this:

```python
t = Tracker(my_emitter)
t.subject.set_platform("mob").set_user_id("user-12345").set_lang("en")
```

We will discuss each of these in turn below:

<a name="set-platform" />

#### 3.1 Change the tracker's platform with `set_platform`

The default platform is "pc". You can change the platform the subject is using by calling:

```python
s.set_platform( {{PLATFORM}} )
```

For example:

```python
s.set_platform("tv") # Running on a Connected TV
```

For a full list of supported platforms, please see the [[Snowplow Tracker Protocol]].

[Back to top](#top)

<a name="set-user-id" />

### 3.2 Set user ID with `set_user_id`

You can set the user ID to any string:

```python
s.set_user_id( "{{USER ID}}" )
```

Example:

```python
s.set_user_id("alexd")
```

[Back to top](#top)

<a name="set-screen-resolution" />

### 3.3 Set screen resolution with `set_screen_resolution`

If your Python code has access to the device's screen resolution, then you can pass this in to Snowplow too:

```python
s.set_screen_resolution( {{WIDTH}}, {{HEIGHT}} )
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```python
s.set_screen_resolution(1366, 768)
```

[Back to top](#top)

<a name="set-viewport-dimensions" />

### 3.4 Set viewport dimensions with `set_viewport`

If your Python code has access to the viewport dimensions, then you can pass this in to Snowplow too:

```python
s.set_viewport( {{WIDTH}}, {{HEIGHT}} )
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```python
s.set_viewport(300, 200)
```

[Back to top](#top)

<a name="set-color-depth" />

### 3.5 Set color depth with `set_color_depth`

If your Python code has access to the bit depth of the device's color palette for displaying images, then you can pass this in to Snowplow too:

```python
s.set_color_depth( {{BITS PER PIXEL}} )
```

The number should be a positive integer, in bits per pixel. Example:

```python
s.set_color_depth(32)
```

[Back to top](#top)

<a name="set-timezone" />

### 3.6 Set timezone with `set_timezone`

This method lets you pass a user's timezone in to Snowplow:

```python
s.timezone( {{TIMEZONE}} )
```

The timezone should be a string:

```python
s.set_color_depth("Europe/London")
```

[Back to top](#top)

<a name="set-lang" />

### 3.7 Set the language with `set_lang`

This method lets you pass a user's language in to Snowplow:

```python
s.set_lang( {{LANGUAGE}} )
```

The language should be a string:

```python
s.set_lang('en')
```

[Back to top](#top)

<a name="multiple-subjects" />

### 3.7 Tracking multiple subjects

You may want to track more than one subject concurrently. To avoid data about one subject being added to events pertaining to another subject, create two subject instances and switch between them using `Tracker.set_subject`:

```python
from snowplow_tracker import Subject, Emitter, Tracker

# Create a simple Emitter which will log events to http://d3rkrsqld9gmqf.cloudfront.net/i
e = Emitter("d3rkrsqld9gmqf.cloudfront.net")

# Create a Tracker instance
t = Tracker(emitters=e, namespace="cf", app_id="CF63A")

# Create a Subject corresponding to a pc user
s1 = Subject()

# Set some data for that user
s1.set_platform("pc")
s1.set_user_id("0a78f2867de")

# Set s1 as the tracker subject
# All events fired will have the information we set about s1 attached
t.set_subject(s1)

# Track user s1 viewing a page
t.track_page_view("http://www.example.com")

# Create another Subject instance corresponding to a mobile user
s2 = Subject()

# All methods of the Subject class return the Subject instance so methods can be chained:
s2.set_platform("mob").set_user_id("0b08f8be3f1")

# Change the tracker subject from s1 to s2
# All events fired will have instead have information we set about s2 attached
t.set_subject(s2)

# Track user s2 viewing a page
t.track_page_view("http://www.example.com")

# Switch back to s1 and track a structured event, this time using method chaining:
t.set_subject(s1).track_struct_event("Ecomm", "add-to-basket", "dog-skateboarding-video", "hd", 13.99)
```

<a name="events" />

## 4. Tracking specific events

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your websites and apps. We are constantly growing the range of functions available in order to capture that data more richly.

Tracking methods supported by the Python Tracker at a glance:

| **Function**                                  | **Description**                                        |
|----------------------------------------------:|:-------------------------------------------------------|
| [`track_page_view()`](#page-view)             | Track and record views of web pages.                   |
| [`track_ecommerce_transaction()`](#ecommerce-transaction)   | Track an ecommerce transaction           |
| [`track_screen_view()`](#screen-view)         | Track the user viewing a screen within the application |
| [`track_struct_event()`](#struct-event)       | Track a Snowplow custom structured event               |
| [`track_unstruct_event()`](#unstruct-event)   | Track a Snowplow custom unstructured event             |

<a name="common" />

### 4.1 Common

All events are tracked with specific methods on the tracker instance, of the form `track_XXX()`, where `XXX` is the name of the event to track.

<a name="custom-contexts" />

### 4.1.1 Custom contexts

In short, custom contexts let you add additional information about the circumstances surrounding an event in the form of a Python dictionary object. Each tracking method accepts an additional optional contexts parameter after all the parameters specific to that method:

```python
def track_page_view(self, page_url, page_title=None, referrer=None, context=None, tstamp=None):
```

The `context` argument should consist of an array of one or more Python dictionaries. The format of each dictionary is the same as for an [unstructured event](#unstruct-event).

**Important:** Even if only one custom context is being attached to an event, it still needs to be wrapped in an array.

If a visitor arrives on a page advertising a movie, the context dictionary might look like this:


```python
{
  "schema": "iglu:com.acme_company/movie_poster/jsonschema/2-1-1",
  "data": {
    "movie_name": "Solaris",
    "poster_country": "JP",
    "poster_year": new Date(1978, 1, 1)
  }
}
```

This is how to fire a page view event with the above custom context:

```python
t.track_page_view("http://www.films.com", "Homepage", context=[{
  "schema": "iglu:com.acme_company/movie_poster/jsonschema/2-1-1",
  "data": {
    "movie_name": "Solaris",
    "poster_country": "JP",
    "poster_year": new Date(1978, 1, 1)
  }
}])
```

Note that even though there is only one custom context attached to the event, it still needs to be placed in an array.

Note also that you should not pass in an empty array of contexts as this will fail validation. Instead of an empty array you can pass in `None`.

<a name="tstamp-arg" />

### 4.1.2 Optional timestamp argument

Each `track...()` method supports an optional timestamp as its final argument; this allows you to manually override the timestamp attached to this event. The timestamp should be in milliseconds since the Unix epoch.

If you do not pass this timestamp in as an argument, then the Python Tracker will use the current time to be the timestamp for the event.

Here is an example tracking a structured event and supplying the optional timestamp argument. We can explicitly supply `None`s for the intervening arguments which are empty:

```python
t.track_struct_event("some cat", "save action", None, None, None, 1368725287000)
```

Alternatively, we can use the argument name:

```python
t.track_struct_event("some cat", "save action", tstamp=1368725287000)
```

Timestamp is counted in milliseconds since the Unix epoch - the same format as generated by `time.time() * 1000`.

<a name="return-values" />

### 4.1.3 Tracker method return values

All tracker methods will return the tracker instance, allowing tracker methods to be chained:

```python
e = AsyncEmitter("d3rkrsqld9gmqf.cloudfront.net")
t = Tracker(e)

t.track_page_view("http://www.example.com").track_screen_view("title screen")
```

[Back to top](#top)

<a name="screen-view" />

### 4.2 Track screen views with `track_screen_view()`

Use `track_screen_view()` to track a user viewing a screen (or equivalent) within your app. Arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `name`       | Human-readable name for this screen | No           | Non-empty string         |
| `id_`         | Unique identifier for this screen  | No            | String                  |
| `context`    | Custom context for the event        | No            | List                    |
| `tstamp`     | When the screen was viewed          | No            | Positive integer        |

Although name and id_ are not individually required, at least one must be provided or the event will fail validation.

Example:

```python
t.track_screen_view("HUD > Save Game", "screen23", null, 1368725287000)
```

[Back to top](#top)

<a name="page-view" />

### 4.3 Track pageviews with `track_page_view()`

Use `track_page_view()` to track a user viewing a page within your app.
Arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `page_url`   | The URL of the page                 | Yes           | Non-empty string        |
| `page_title` | The title of the page               | No            | String                  |
| `referrer`   | The address which linked to the page| No            | String                  |
| `context`    | Custom context for the event        | No            | List                    |
| `tstamp`     | When the pageview occurred          | No            | Positive integer        |

Example:

```python
t.track_page_view("www.example.com", "example", "www.referrer.com")
```

[Back to top](#top)

<a name="ecommerce-transaction" />

### 4.4 Track ecommerce transactions with `track-ecommerce-transaction()`

Use `track_ecommerce_transaction()` to track an ecommerce transaction.
Arguments:

| **Argument**     | **Description**                   | **Required?** | **Validation**    |
|-----------------:|:----------------------------------|:--------------|:------------------|
| `order_id`    | ID of the eCommerce transaction      | Yes           | Non-empty string  |
| `total_value` | Total transaction value              | Yes           | Int or Float      |
| `affiliation` | Transaction affiliation              | No            | String            |
| `tax_value`   | Transaction tax value                | No            | Int or Float      |
| `shipping`    | Delivery cost charged                | No            | Int or Float      |
| `city`        | Delivery address city                | No            | String            |
| `state`       | Delivery address state               | No            | String            |
| `country`     | Delivery address country             | No            | String            |
| `currency     | Transaction currency                 | No            | String            |
| `items`       | Items in the transaction             | Yes           | List              |
| `context`     | Custom context for the event         | No            | List              |
| `tstamp`      | When the transaction event occurred  | No            | Positive integer  |

The `items` argument is an array of Python dictionaries representing the items in the transaction. `track_ecommerce_transaction` fires multiple events: one "transaction" event for the transaction as a whole, and one "transaction item" event for each element of the `items` array. Each transaction item event will have the same timestamp, order_id, and currency as the main transaction event.

These are the fields that can appear in a transaction item dictionary:

| **Field**       | **Description**                     | **Required?** | **Validation**           |
|----------------:|:------------------------------------|:--------------|:-------------------------|
| `"sku"`         | Item SKU                            | Yes           | Non-empty string         |
| `"price"`       | Item price                          | Yes           | Int or Float             |
| `"quantity"`    | Item quantity                       | Yes           | Int                      |
| `"name"`        | Item name                           | No            | String                   |
| `"category"`    | Item category                       | No            | String                   |
| `"context"`     | Custom context for the event        | No            | List                     |

Example of tracking a transaction containing two items:

```python
t.track_ecommerce_transaction("6a8078be", 35, city="London", currency="GBP", items=
    [{
        "sku": "pbz0026",
        "price": 20,
        "quantity": 1
    },
    {
        "sku": "pbz0038",
        "price": 15,
        "quantity": 1
    }])
```

[Back to top](#top)

<a name="ecommerce-transaction-item" />

### 4.5 Track ecommerce transactions with `track_ecommerce_transaction_item()`

Use `track_ecommerce_transaction_item()` to track an individual line item within an ecommerce transaction.

Arguments:

| **Argument**  | **Description**                     | **Required?** | **Validation**           |
|--------------:|:------------------------------------|:--------------|:-------------------------|
| `id`          | Order ID                            | Yes           | Non-empty string         |
| `sku`         | Item SKU                            | Yes           | Non-empty string         |
| `price`       | Item price                          | Yes           | Int or Float             |
| `quantity`    | Item quantity                       | Yes           | Int                      |
| `name`        | Item name                           | No            | String                   |
| `category`    | Item category                       | No            | String                   |
| `context`     | Custom context for the event        | No            | List                     |
| `tstamp`      | When the transaction event occurred | No            | Positive integer         |

Example:

```python
t.track_ecommerce_transaction_item("order-789", "2001", 49.99, 1, "Green shoes", "clothing")
```

[Back to top](#top)

<a name="struct-event" />

### 4.6 Track structured events with `track_struct_event()`

Use `track_struct_event()` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument** | **Description**                                                  | **Required?** | **Validation**           |
|-------------:|:---------------------------------------------------------------  |:--------------|:-------------------------|
| `category`   | The grouping of structured events which this `action` belongs to | Yes           | Non-empty string         |
| `action`     | Defines the type of user interaction which this event involves   | Yes           | Non-empty string         |
| `label`      | A string to provide additional dimensions to the event data      | No            | String                   |
| `property`   | A string describing the object or the action performed on it     | No            | String                   |
| `value`      | A value to provide numerical data about the event                | No            | Int or Float             |
| `context`    | Custom context for the event                                     | No            | List                     |
| `tstamp`     | When the structured event occurred                               | No            | Positive integer         |

Example:

```python
t.track_struct_event("shop", "add-to-basket", None, "pcs", 2)
```

[Back to top](#top)

<a name="unstruct-event" />

### 4.7 Track unstructured events with `track_unstruct_event()`

Use `track_unstruct_event()` to track a custom event which consists of a name and an unstructured set of properties. This is useful when:

* You want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow), or
* You want to track events which have unpredictable or frequently changing properties

The arguments are as follows:

| **Argument**   | **Description**                      | **Required?** | **Validation**          |
|---------------:|:-------------------------------------|:--------------|:------------------------|
| `event_json`   | The properties of the event          | Yes           | Dict                    |
| `context`      | Custom context for the event         | No            | List                    |
| `tstamp`       | When the unstructured event occurred | No            | Positive integer        |

Example:

```python
t.track_unstruct_event({
  "schema": "com.example_company/save-game/jsonschema/1-0-2",
  "data": {
    "save_id": "4321",
    "level": 23,
    "difficultyLevel": "HARD",
    "dl_content": true
  }
})
```

The `event_json` must be a Python dictionary with two fields: `schema` and `data`. `data` is a flat dictionary containing the properties of the unstructured event. `schema` identifies the JSON schema against which `data` should be validated.

For more on JSON schema, see the [blog post][self-describing-jsons].

<a name="emitter" />

## 5. Emitters

Tracker instances must be initialized with an emitter. This section will go into more depth about the Emitter class and its subclasses.

<a name="base-emitter" />

## 5.1 The basic Emitter class

At its most basic, the Emitter class only needs a collector URI:

```python
from snowplow_tracker import Emitter

e = Emitter("d3rkrsqld9gmqf.cloudfront.net")
```

This is the signature of the constructor for the base Emitter class:

```python
def __init__(self, endpoint,
             protocol="http", port=None, method="get",
             buffer_size=None, on_success=None, on_failure=None):
```

| **Argument**   | **Description**                      | **Required?** | **Validation**          |
|---------------:|:-------------------------------------|:--------------|:------------------------|
| `endpoint`     | The collector URI                    | Yes           | Dict                    |
| `protocol`     | Request protocol: HTTP or HTTPS      | No            | List                    |
| `port`         | The port to connect to               | No            | Positive integer        |
| `method`       | The method to use: "get" or "post"       | No            | String        |
| `buffer_size`  | Number of events to store before flushing | No            | Positive integer        |
| `on_success`   | Callback executed when a flush is successful | No            | Function taking 1 argument        |
| `on_failure`   | Callback executed when a flush is unsuccessful | No            | Function taking 2 arguments        |

`protocol` defaults to "http" but can also be "https".

When the emitter receives an event, it adds it to a buffer. When the queue is full, all events in the queue get sent to the collector. The `buffer_size` argument allows you to customize the queue size. By default, it is 1 for GET requests and 10 for POST requests. (So in the case of GET requests, each event is fired as soon as the emitter receives it.) If the emitter is configured to send POST requests, then instead of sending one for every event in the buffer, it will send a sing request containing all those events in JSON format.

*Warning: `method` defaults to GET because Snowplow collectors do not currently support POST requests.*

`on_success` is an optional callback that will execute whenever the queue is flushed successfully, that is, whenever every request sent has status code 200. It will be passed one argument: the number of events that were sent.

`on_failure` is similar, but executes when the flush is not wholly successful. It will be passed two arguments: the number of events that were successfully sent, and an array of unsent requests. (If the emitter is configured to send POST requests, the array will actually be a string, but it can be turned back into an array of Python dictionaries (each corresponding to an event) by using `json.loads`.)

An example:

```python
def f(x):
    print(str(x) + " events sent successfully!")

unsent_events = []

def g(x, y):
    print(str(x) + " events sent successfully!")
    print("These events were not sent successfully and have been stored in unsent_events:")
    for event_dict in y:
        print(event_dict)
        unsent_events.append(event_dict)

e = Emitter("d3rkrsqld9gmqf.cloudfront.net", buffer_size=3, on_success=f, on_failure=g)

t = Tracker(e)

# This doesn't cause the emitter to send a request because the buffer_size was set to 3, not 1
t.track_page_view("http://www.example.com")
t.track_page_view("http://www.example.com/page1")

# This does cause the emitter to try to send all 3 events
t.track_page_view("http://www.example.com/page2")

# Since the method is GET by default, 3 separate requests are sent
# If any of them are unsuccessful, they will be stored in the unsent_events variable
```

<a name="async-emitter" />

## 5.2 The AsyncEmitter class

```python
from snowplow_tracker import AsyncEmitter

e = AsyncEmitter("d3rkrsqld9gmqf.cloudfront.net")
```

The `AsyncEmitter` class works just like the Emitter class. It has one advantage, though: HTTP(S) requests are sent asynchronously, so the Tracker won't be blocked while the Emitter waits for a response. For this reason, the AsyncEmitter is recommended over the base `Emitter` class.

<a name="celery-emitter" />

## 5.3 The CeleryEmitter class

The `CeleryEmitter` class works just like the base `Emitter` class, but it registers sending requests as a task for a [Celery][celery] worker. If there is a module named snowplow_celery_config.py on your PYTHONPATH, it will be used as the Celery configuration file; otherwise, a default configuration will be used. You can run the worker using this command:

```bash
celery -A snowplow_tracker.emitters worker --loglevel=debug
```

Note that `on_success` and `on_failure` callbacks cannot be supplied to this emitter.

<a name="redis-emitter" />

## 5.4 The RedisEmitter class

Use a RedisEmitter instance to store events in a [Redis][redis] database for later use. This is the RedisEmitter constructor function:

```python
def __init__(self, rdb=None, key="snowplow"):
```

`rdb` should be an instance of either the `Redis` or `StrictRedis` class, found in the `redis` module. If it is not supplied, a default will be used. `key` is the key used to store events in the database. It defaults to "snowplow". The format for event storage is a Redis list of JSON strings.

An example:

```python
from snowplow_tracker import RedisEmitter, Tracker
import redis

rdb = redis.StrictRedis(db=2)

e = RedisEmitter(rdb, "my_snowplow_key")

t = Tracker(e)

t.track_page_view("http://www.example.com")

# Check that the event was stored in Redis:
print(rdb.lrange("my_snowplowkey", 0, -1))
# prints something like:
# ['{"tv":"py-0.4.0", "ev": "pv", "url": "http://www.example.com", "dtm": 1400252420261, "tid": 7515828, "p": "pc"}']
```

<a name="manual-flushing" />

### 5.5 Manual flushing

You can flush the emitter manually using the `flush` method of the `Tracker` instance which is sending events to the emitter. This is a blocking call which synchronously sends all events in the emitter's buffer. In the case of the `AsyncEmitter`, it also forces all running threads to finish within 10 seconds.

```python
t.flush()
```

<a name="multiple-emitters" />

### 5.6 Multiple emitters

You can configure a tracker instance to send events to multiple emitters by passing the `Tracker` constructor function an array of emitters instead of a single emitter, or by using the `addEmitter` method:

```python
from snowplow_tracker import Subject, Tracker, AsyncEmitter, RedisEmitter
import redis

e1 = AsyncEmitter("collector1.cloudfront.net", method="get")
e1 = AsyncEmitter("collector2.cloudfront.net", method="post")

t = Tracker([e1, e2])

rdb = redis.StrictRedis(db=2)

e3 = RedisEmitter(rdb, "my_snowplow_key")

t.addEmitter(e3)
```

<a name="custom-emitters" />

### 5.7 Custom emitters

You can create your own custom emitter class, either from scratch or by subclassing one of the existing classes (with the exception of `CeleryEmitter`, since it uses the `pickle` module which doesn't work correctly with a class subclassed from a class located in a different module). The only requirement for compatibility is that is must have an `input` method which accepts a Python dictionary of name-value pairs.

<a name="contracts" />

## 6. Contracts

Python is a dynamically typed language, but each of our methods expects its arguments to be of specific types and value ranges, and validates that to be the case. These checks are done using the [PyContracts][pycontracts] library.

If the validation check fails, then a runtime error is thrown:

```python
s = Subject()
t.set_color_depth("walrus")
```
```
contracts.interface.ContractNotRespected: Breach for argument 'depth' to Subject:set_color_depth().
Expected type 'int', got 'str'.
checking: Int      for value: Instance of str: 'walrus'
checking: $(Int)   for value: Instance of str: 'walrus'
checking: int      for value: Instance of str: 'walrus'
Variables bound in inner context:
- self: Instance of Tracker: <snowplow_tracker.tracker.Tracker object...> [clip]

```

If your value is of the wrong type, convert it before passing it into the `track...()` method, for example:

```python
level_idx = 42
t.track_screen_view("Game Level", str(level_idx))
```

You can turn off type checking to improve performance like this:

```python
from snowplow_tracker import disable_contracts
disable_contracts()
```

[Back to top](#top)

<a name="logging" />

## 7. Logging

The emitters.py module has Python logging turned to give you information about requests being sent. The logger prints messages about what emitters are doing. By default, only messages with priority "INFO" or higher will be logged.

To change this:

```python
from snowplow_tracker import logger

# Log all messages, even DEBUG messages
logger.setLevel(10)

# Log only messages with priority WARN or higher
logger.setLevel(30)

# Turn off all logging
logger.setLevel(60)
```

[Back to top](#top)

<a name="redis-worker" />

## 8. The RedisWorker class

The tracker comes with a RedisWorker class which sends Snowplow events from Redis to an emitter. The RedisWorker constructor is similar to the RedisEmitter constructor:

```python
def __init__(self, _consumer, key=None, dbr=None):
```

This is how it is used:

```python
from snowplow_tracker import AsyncEmitter
from snowplow_tracker.redis_worker import RedisWorker

e = Emitter("d3rkrsqld9gmqf.cloudfront.net")

r = RedisWorker(e, key="snowplow_redis_key")

r.run()
```

This will set up a worker which will run indefinitely, taking events from the Redis list with key "snowplow_redis_key" and inputting them to an AsyncEmitter, which will send them to a cloudfront collector. If the process receives a SIGINT signal (for example, due to a Ctrl-C keyboard interrupt), cleanup will occur before exiting to ensure no events are lost.

[Back to top](#top)

[python-0.2]: https://github.com/snowplow/snowplow/wiki/Python-Tracker-v0.2
[python-0.3]: https://github.com/snowplow/snowplow/wiki/Python-Tracker-v0.3
[python-0.4]: https://github.com/snowplow/snowplow/wiki/Python-Tracker-v0.4
[python-0.6]: https://github.com/snowplow/snowplow/wiki/Python-Tracker-v0.6
[python-0.7]: https://github.com/snowplow/snowplow/wiki/Python-Tracker-v0.7
[python-latest]: https://github.com/snowplow/snowplow/wiki/Python-Tracker
[pycontracts]: http://andreacensi.github.io/contracts/


[jsonschema]: http://snowplowanalytics.com/blog/2014/05/13/introducing-schemaver-for-semantic-versioning-of-schemas/
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
[base64]: https://en.wikipedia.org/wiki/Base64
[celery]: http://www.celeryproject.org/
[redis]: http://redis.io/
