<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow technical documentation) > [**Trackers**](trackers) > Ruby Tracker

**This page refers to version 0.4.1 of the Snowplow Ruby Tracker. Documentation for other versions is available:**

*[Version 0.2][ruby-0.2]*  
*[Version 0.3][ruby-0.3]*  
*[Version 0.5][ruby-0.5]*  
*[Version 0.6][ruby-latest]*  

**Please note** that this version of the Ruby Tracker is dependent upon the [Snowplow 0.9.14 release][snowplow-0.9.14]. You will need to be running version 0.9.14 or later of Snowplow for events sent by the tracker using POST to be successfully processed. Snowplow 0.9.14 contains updates to the Hadoop Enrich and Scala Hadoop Shred jobs to allow the newer self-describing JSON version which the Ruby Tracker sends for POSTs. For more information, please refer to tickets [#1220][issue-1220] and [#1231][issue-1231].

## Contents

- 1. [Overview](#overview)  
- 2. [Initialization](#init)  
  - 2.1 [Requiring the module](#requiring)
  - 2.2 [Creating a tracker](#create-tracker)  
  - 2.3 [Creating multiple trackers](#multi-tracker)
- 3 [Adding extra data](#add-data)
  - 3.1 [`set_platform`](#set-platform)
  - 3.2 [`set_user_id`](#set-user-id)
  - 3.3 [`set_screen_resolution`](#set-screen-resolution)
  - 3.4 [`set_viewport`](#set-viewport)
  - 3.5 [`set_color_depth`](#set-color-depth)
  - 3.6 [`set_timezone`](#set-timezone)
  - 3.7 [`set_lang`](#set-language)
  - 3.8 [`set_ip_address`](#set-ip-address)
  - 3.9 [`set_useragent`](#set-useragent)
  - 3.10 [`set_domain_user_id`](#set-domain-user-id)
  - 3.11 [`set_network_user_id`](#set-network-user-id)
- 4. [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Argument validation](#validation)
    - 4.1.2 [Optional context argument](#context-arg)
    - 4.1.3 [Optional timestamp argument](#tstamp-arg)
    - 4.1.4 [Example](#common-example)
  - 4.2 [`track_screen_view`](#screen-view)
  - 4.3 [`track_page_view`](#page-view)
  - 4.4 [`track_ecommerce_transaction`](#ecommerce-transaction)
  - 4.5 [`track_struct_event`](#struct-event)
  - 4.6 [`track_unstruct_event`](#unstruct-event)
- 5. [Emitters](#emitters)
  - 5.1 [Overview](#emitters-overview)
  - 5.2 [The AsyncEmitter class](#async-emitter)
  - 5.3 [Multiple emitters](#multiple-emitters)
  - 5.4 [Manual flushing](#flushing)
- 6 [Contracts](#contracts)
- 7 [Logging](#logging)
- 8 [Advanced usage](#advanced)
  - 8.1 [snowplow_ruby_duid](#snowplow-ruby-duid)

<a name="overview" />
## 1. Overview

The Snowplow Ruby Tracker allows you to track Snowplow events in your Ruby applications and gems and Ruby on Rails web applications.

The tracker should be straightforward to use if you are comfortable with Ruby development; any prior experience with Snowplow's [[Python Tracker]], [[JavaScript Tracker]], [[Lua Tracker]], Google Analytics or Mixpanel (which have similar APIs to Snowplow) is helpful but not necessary.

The Ruby Tracker and Python Tracker have very similiar functionality and APIs.

There are three main classes which the Ruby Tracker uses: subjects, emitters, and trackers.

A subject represents a single user whose events are tracked, and holds data specific to that user. If your tracker will only be tracking a single user, you don't have to create a subject - it will be done automatically.

A tracker always has one active subject at a time associated with it. It constructs events with that subject and sends them to one or more emitters, which sends them on to a Snowplow collector.

<a name="init" />
## 2. Initialization

Assuming you have completed the [[Ruby Tracker Setup]] for your Ruby project, you are ready to initialize the Ruby Tracker.

<a name="requiring" />
### 2.1 Requiring the module

Require the Ruby Tracker into your code like this:

```ruby
require 'snowplow_tracker'
```

You can now initialize tracker instances.

<a name="create-tracker" />
### 2.2 Creating a tracker

Initialize a tracker instance like this:

```ruby
emitter = SnowplowTracker::Emitter.new("d3rkrsqld9gmqf.cloudfront.net")
tracker = SnowplowTracker::Tracker.new(e)
```

If you wish to send events to more than one emitter, you can provide an array of emitters to the tracker constructor.

This tracker will log events to http://d3rkrsqld9gmqf.cloudfront.net/i. There are four other optional parameters:

```ruby
def initialize(endpoint, subject=nil, namespace=nil, app_id=nil, encode_base64=true)
```

`subject` is a subject with which the tracker is initialized.

`namespace` is a name for the tracker which will be added to every event the tracker fires. This is useful if you have initialized more than one tracker. `app_id` is the unique ID for the Ruby application. `encode_base64` determines whether JSONs in the querystring for an event will be base64-encoded.

So a more complete tracker initialization example might look like this:

```ruby
initial_subject = SnowplowTracker::Subject.new
emitter = SnowplowTracker::Emitter.new("d3rkrsqld9gmqf.cloudfront.net")
tracker = SnowplowTracker::Tracker.new(emitter, initial_subject, 'cf', 'ID-ap00035', false)
```

<a name="multi-tracker" />
### 2.3 Creating multiple trackers

Each tracker instance is completely sandboxed, so you can create multiple trackers as you see fit.

Here is an example of instantiating two separate trackers:

```ruby
t1 = SnowplowTracker::Tracker.new(SnowplowTracker::AsyncEmitter.new("d3rkrsqld9gmqf.cloudfront.net"), nil, "t1")
t1.set_platform("cnsl")
t1.track_page_view("http://www.example.com")

t2 = SnowplowTracker::Tracker.new(SnowplowTracker::AsyncEmitter.new("my-company.c.snplow.com"), nil, "t2")
t2.set_platform("cnsl")
t2.track_screen_view("Game HUD", "23")

t1.track_screen_view("Test", "23") # Back to first tracker 
```

[Back to top](#top)

[Back to top](#top)

<a name="add-data" />
## 3. Adding extra data

You can configure the a tracker instance with additional information about your application's environment or current user. This data will be attached to every event the tracker fires regarding the subject. Here are the available methods:

| **Function**                                      | **Description**                                        |
|--------------------------------------------------:|:-------------------------------------------------------|
| [`set_platform`](#set-platform)                   | Set the application platform     |
| [`set_user_id`](#set-user-id)                     | Set the user ID                   |
| [`set_screen_resolution`](#set-screen-resolution) | Set the screen resolution          |
| [`set_viewport`](#set-viewport)                   | Set the viewport dimensions  | 
| [`set_color_depth`](#set-color-depth)             | Set the screen color depth |
| [`set_timezone`](#set-timezone)                   | Set the timezone               |
| [`set_lang`](#set-language)                       | Set the language             |

There are two ways to call these methods:

* Call them on a Subject instance. They will update the data associated with that subject and return the subject.
* Call them on the Tracker instance. They will update the data associated with the currently active subject for that tracker and return the tracker.

For example:

```ruby
s0 = SnowplowTracker::Subject.new
emitter = SnowplowTracker::Emitter.new("d3rkrsqld9gmqf.cloudfront.net")
my_tracker = SnowplowTracker::Tracker.new(emitter, s0)

# The following two lines are equivalent, except that the first returns s0 and the second returns my_tracker
s0.set_platform('mob')
my_tracker.set_platform('mob')
```

If you are using multiple subjects, you can use the `set_subject` tracker method to change which Subject instance is active:

```ruby
s0 = SnowplowTracker::Subject.new
emitter = SnowplowTracker::Emitter.new("d3rkrsqld9gmqf.cloudfront.net")
my_tracker = SnowplowTracker::Tracker.new(emitter, s0)

# Set the viewport for the active subject, s0
my_tracker.set_viewport(300, 500)

# The data associated with s0 will be sent with this event
my_tracker.track_screen_view('title page')

# Create a new subject
s1 = SnowplowTracker::Subject.new

# Make s1 the active subject and set its viewport
my_tracker.set_subject(s1).set_viewport(600,1000)

# The data associated with s0 will be sent with this event
my_tracker.track_screen_view('another page')

# Change the subject back to s0 and track another event
my_tracker.set_subject(s0).track_screen_view('final page')
```

<a name="set-platform" />
### 3.1 Set the tracker's platform with `set_platform`

The platform can be any one of `'pc'`, `'tv'`, `'mob'`, `'cnsl'`, or `'iot'`. The default platform is `'srv'`.

```ruby
tracker.set_platform('mob')
```

<a name="set-user-id" />
### 3.2 Set the user ID with `set_user_id`

You can make the user ID a string of your choice:

```ruby
tracker.set_user_id('user-000563456')
```

<a name="set-screen-resolution" />
### 3.3 Set the screen resolution with `set_screen_resolution`

If your Ruby code has access to the device's screen resolution, you can pass it in to Snowplow. Both numbers should be positive integers; note the order is width followed by height. Example:

```ruby
tracker.set_screen_resolution(1366, 768)
```

<a name="set-viewport" />
### 3.4 Set the viewport dimensions with `set_viewport`

Similarly, you can pass the viewport dimensions in to Snowplow. Again, both numbers should be positive integers and the order is width followed by height. Example:

```ruby
tracker.set_viewport(300, 200)
```

<a name="set-color-depth" />
### 3.5 Set the color depth with `set_color_depth`

If your Ruby code has access to the bit depth of the device's color palette for displaying images, you can pass it in to Snowplow. The number should be a positive integer, in bits per pixel.

```ruby
tracker.set_color_depth(24)
```

<a name="set-timezone" />
### 3.6 Setting the timezone with `set_timezone`

If your Ruby code has access to the timezone of the device, you can pass it in to Snowplow:

```ruby
tracker.set_timezone('Europe London')
```

<a name="set-language" />
### 3.7 Setting the language with `set_lang`

You can set the language field like this:

```ruby
tracker.set_lang('en')
```

<a name="set-ip-address" />
### 3.8 Setting the IP address with `set_ip_address`

If you have access to the user's IP address, you can set it like this:

```ruby
tracker.set_ip_address('34.633.11.139')
```

<a name="set-useragent" />
### 3.9 Setting the useragent with `set_useragent`

If you have access to the user's useragent (sometimes called "browser string"), you can set it like this:

```ruby
tracker.set_useragent('Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0')
```

<a name="set-domain-user-id" />
### 3.10 Setting the domain user ID with `set_domain_user_id`

The `domain_userid` field of the Snowplow event model corresponds to the ID stored in the first party cookie set by the Snowplow JavaScript Tracker. If you want to match up server-side events with client-side events, you can set the domain user ID for server-side events like this:

```ruby
tracker.set_domain_user_id('c7aadf5c60a5dff9')
```

You can extract the domain user ID from the Ruby on Rails `cookies` object like this:

```ruby
def snowplow_cookie
  cookies.find { |(key, value)| key =~ /^_sp_id/ }.last
end

def domain_user_id
  if snowplow_cookie.present?
    snowplow_cookie.split('.').first
  end
end
```

The first argument is the cookies object (see the [documentation](http://api.rubyonrails.org/classes/ActionDispatch/Cookies.html)).

If you used the "cookieName" configuration option of the Snowplow JavaScript Tracker, replace "_sp_" with the same string you passed as the cookieName.

<a name="set-network-user-id" />
### 3.11 Setting the network user ID with `set_network_user_id`

The `network_user_id` field of the Snowplow event model corresponds to the ID stored in the third party cookie set by the Snowplow Clojure Collector. You can set the network user ID for server-side events like this:

```ruby
tracker.set_network_user_id('ecdff4d0-9175-40ac-a8bb-325c49733607')
```

[Back to top](#top)

<a name="events" />
## 4. Tracking specific events

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your websites and apps. We are constantly growing the range of functions available in order to capture that data more richly.

Tracking methods supported by the Ruby Tracker at a glance:

| **Function**                                  | **Description**                                        |
|----------------------------------------------:|:-------------------------------------------------------|
| [`track_page_view`](#page-view)             | Track and record views of web pages.                   |
| [`track_ecommerce_transaction`](#ecommerce-transaction)   | Track an ecommerce transaction          | 
| [`track_screen_view`](#screen-view)         | Track the user viewing a screen within the application |
| [`track_struct_event`](#struct-event)       | Track a Snowplow custom structured event               |
| [`track_unstruct_event`](#unstruct-event)   | Track a Snowplow custom unstructured event             |

<a name="common" />
### 4.1 Common

All events are tracked with specific methods on the tracker instance, of the form `track_XXX()`, where `XXX` is the name of the event to track.

All tracker methods return the tracker instance, and so are chainable.

<a name="validation" />
#### 4.1.1 Argument validation

Each `track_XXX` method expects arguments of a certain type. The types are validated using the Ruby Contracts library. If a check fails, a runtime error is thrown. The section for each `track_XXX` method specifies the expected argument types for that method.

<a name="context-arg" />
#### 4.1.2 Optional context argument

Each `track_XXX` method has `context` as its penultimate optional parameter. This is for an optional nonempty array of [self-describing custom context JSONs][self-describing-jsons] attached to the event. Each element of the `context` argument should be a hash whose keys are "schema", containing a pointer to the JSON schema against which the context will be validated, and "data", containing the context data itself. The "data" field should contain a flat hash of key-value pairs. 

**Important:**
* Even if only one custom context is being attached to an event, it still needs to be wrapped in an array.
* If you do provide the argument it shouldn't be an empty array. Pass in `nil` instead of an empty array. Otherwise the context will fail validation.

For example, an array containing two custom contexts relating to the event of a movie poster being viewed:

```ruby
# Array of contexts
[{
  # First context
  'schema' => 'iglu:com.my_company/movie_poster/jsonschema/1-0-0',
  'data' => {
    'movie_name' => 'Solaris',
    'poster_country' => 'JP',
    'poster_year$dt' => new Date(1978, 1, 1)  
  }
},
{
  # Second context
  'schema' => 'iglu:com.my_company/customer/jsonschema/1-0-0',
  'data' => {
      'p_buy' => 0.23,
      'segment' => 'young adult'
  }
}]
```

The keys of a context hash can be either strings or Ruby symbols.

For more on how to use custom contexts, see the [blog post][contexts] which introduced them.

<a name="tstamp-arg" />
#### 4.1.3 Optional timestamp argument

After the optional context argument, each `track_XXX` method supports an optional timestamp as its final argument. This allows you to manually override the timestamp attached to this event. If you do not pass this timestamp in as an argument, then the Ruby Tracker will use the current time to be the timestamp for the event. Timestamp is counted in milliseconds since the Unix epoch - the same format generated by `Time.now.to_i * 1000` in Ruby.

<a name="common-example" />
#### 4.1.4 Example

Here is an example of a page view event with custom context and timestamp arguments supplied:

```ruby
tracker.track_page_view('http://www.film_company.com/movie_poster', nil, nil, [{
  # First context
  'schema' => 'iglu:com.my_company/movie_poster/jsonschema/1-0-0',
  'data' => {
    'movie_name' => 'Solaris',
    'poster_country' => 'JP',
    'poster_year$dt' => new Date(1978, 1, 1)  
  }
},
{
  # Second context
  'schema' => 'iglu:com.my_company/customer/jsonschema/1-0-0',
  'data' => {
      'p_buy' => 0.23,
      'segment' => 'young adult'
  }
}], 1368725287000)
```

<a name="screen-view" />
### Track screen views with `track_screen_view`

Use `track_screen_view()` to track a user viewing a screen (or equivalent) within your app. Arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `name`       | Human-readable name for this screen | Yes           | String                  |
| `id`         | Unique identifier for this screen   | No            | String                  |
| `context`    | Custom context                      | No            | Array[Hash]                    |
| `tstamp`     | When the screen was viewed          | No            | Positive integer        |

Example:

```ruby
tracker.track_screen_view("HUD > Save Game", "screen23")
```

<a name="page-view" />
### Track page views with `track_page_view`

Use `track_page_view()` to track a user viewing a page within your app.
Arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `page_url`   | The URL of the page                 | Yes           | String                  |
| `page_title` | The title of the page               | No            | String                  |
| `referrer`   | The address which linked to the page| No            | String                  |
| `context`    | Custom context                      | No            | Array[Hash]                    |
| `tstamp`     | When the pageview occurred          | No            | Positive integer        |

Example:

```ruby
t.track_page_view("www.example.com", "example", "www.referrer.com")
```

<a name="ecommerce-transaction" />
### 4.4 Track ecommerce transactions with `track-ecommerce-transaction()`

Use `track_ecommerce_transaction()` to track an ecommerce transaction.
Arguments:

| **Argument**     | **Description**                      | **Required?** | **Validation**       |
|-----------------:|:-------------------------------------|:--------------|:---------------------|
| `transaction`    | Data for the whole transaction       | Yes           | Hash                 |
| `items`          | Data for each item                   | Yes           | Array of hashes      |
| `context`        | Custom context                       | No            | Array[Hash]                 |
| `tstamp`         | When the transaction event occurred  | No            | Positive integer     |

The `transaction` argument is a hash containing information about the transaction. Here are the fields supported in this hash:

| **Field**     | **Description**                      | **Required?** | **Validation**       |
|--------------:|:-------------------------------------|:--------------|:---------------------|
| `order_id`    | ID of the eCommerce transaction      | Yes           | String               |
| `total_value` | Total transaction value              | Yes           | Int or Float         |
| `affiliation` | Transaction affiliation              | No            | String               |
| `tax_value`   | Transaction tax value                | No            | Int or Float         |
| `shipping`    | Delivery cost charged                | No            | Int or Float         |
| `city`        | Delivery address city                | No            | String               |
| `state`       | Delivery address state               | No            | String               |
| `country`     | Delivery address country             | No            | String               |
| `currency`    | Transaction currency                 | No            | String               |

The transaction parameter might look like this:

```ruby
{
  'order_id' => '12345'
  'total_value' => 35
  'city' => 'London'
  'country' => 'UK'
  'currency' => 'GBP'
}
```

The `items` parameter is an array of hashes. Each hash represents one item in the transaction. Here are the fields supported for each item:

| **Argument**   | **Description**                     | **Required?** | **Validation**           |
|---------------:|:------------------------------------|:--------------|:-------------------------|
| `sku`          | Item SKU                            | Yes           | String                   |
| `price`        | Item price                          | Yes           | Int or Float             |
| `quantity`     | Item quantity                       | Yes           | Int                      |
| `name`         | Item name                           | No            | String                   |
| `category`     | Item category                       | No            | String                   |
| `context`      | Custom context                      | No            | Array[Hash]                     |

The `items` parameter might look like that:

```ruby
[{
  'sku' => 'pbz0026',
  'price' => 20,
  'quantity' => 1,
  'category' => 'film'
},
{
  'sku' => 'pbz0038',
  'price' => 15,
  'quantity' => 1,
  'name' => 'red shoes'
}]
```

The whole method call would look like this:

```ruby
tracker.track_ecommerce_transaction({
  'order_id' => '12345'
  'total_value' => 35
  'city' => 'London'
  'country' => 'UK'
  'currency' => 'GBP'
},
[{
  'sku' => 'pbz0026',
  'price' => 20,
  'quantity' => 1,
  'category' => 'film'
},
{
  'sku' => 'pbz0038',
  'price' => 15,
  'quantity' => 1,
  'name' => 'red shoes'
}])
```

This will fire three events: one for the transaction as a whole, which will include the fields in the `transaction` argument, and one for each item. The `order_id` and `currency` fields in the `transaction` argument will also be attached to each the items' events.

All three events will have the same timestamp and same randomly generated Snowplow transaction ID.

Note that each item in the transaction can have its own custom context.

<a name="struct-event" />
### 4.5 Track structured events with `track_struct_event`

Use `track_struct_event()` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument** | **Description**                                                  | **Required?** | **Validation**           |
|-------------:|:---------------------------------------------------------------  |:--------------|:-------------------------|
| `category`   | The grouping of structured events which this `action` belongs to | Yes           | String                   |
| `action`     | Defines the type of user interaction which this event involves   | Yes           | String                   |
| `label`      | A string to provide additional dimensions to the event data      | No            | String                   |
| `property`   | A string describing the object or the action performed on it     | No            | String                   |
| `value`      | A value to provide numerical data about the event                | No            | Int or Float             |
| `context`    | Custom context                                                   | No            | Array[Hash]                     |
| `tstamp`     | When the structured event occurred                               | No            | Positive integer         |


Example:

```ruby
tracker.track_struct_event("shop", "add-to-basket", nil, "pcs", 2)
```

<a name="unstruct-event" />
### 4.6 Track unstructured events with ```track_unstruct_event```

Use `track_unstruct_event()` to track a custom event which consists of a name and an unstructured set of properties. This is useful when:

* You want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow), or
* You want to track events which have unpredictable or frequently changing properties

The arguments are as follows:

| **Argument**   | **Description**                      | **Required?** | **Validation**          |
|---------------:|:-------------------------------------|:--------------|:------------------------|
| `event_json`   | The properties of the event          | Yes           | Hash                    |
| `context`      | Custom context                       | No            | Array[Hash]                    |
| `tstamp`       | When the unstructured event occurred | No            | Positive integer        |

Example:

```ruby
tracker.track_unstruct_event({
  "schema" => "com.example_company/save_game/jsonschema/1-0-2",
  "data" => {
    "saveId" => "4321",
    "level" => 23,
    "difficultyLevel" => "HARD",
    "dlContent" => true 
  }
})
```

The `event_json` argument is [self-describing JSON][self-describing-jsons]. It has two fields: "schema", containing a pointer to the JSON schema for the event, and "data", containing the event data itself. The data field must be flat: properties cannot be nested.

The keys of the `event_json` hash can be either strings or Ruby symbols.

[Back to top](#top)

<a name="emitters" />
## 5. Emitters

Tracker instances must be initialized with an emitter. This section will go into more depth about the Emitter and AsyncEmitter classes.

<a name="emitters-overview" />
### 5.1. Overview

Each tracker instance must now be initialized with an Emitter which is responsible for firing events to a Collector. An Emitter instance is initialized with two arguments: an endpoint and an optional configuration hash. 

A simple example with just an endpoint:


```ruby
# Create an emitter
my_emitter = SnowplowTracker::Emitter.new('d3rkrsqld9gmqf.cloudfront.net')
```

A complicated example using every setting:

```ruby
# Create an emitter
my_emitter = SnowplowTracker::Emitter.new('d3rkrsqld9gmqf.cloudfront.net', {
  :protocol => 'https',
  :method => 'post',
  :port => 80,
  :buffer_size => 0,
  :on_success => lambda { |success_count| 
    puts '#{success_count} events sent successfully'
  },
  :on_failure => lambda { |success_count, failures| 
    puts '#{success_count} events sent successfully, #{failures.size} events sent unsuccessfully'
  }
})
```

Every setting in the configuration hash is optional. Here is what they do:

* `:protocol` determines whether events will be sent using HTTP or HTTPS. It defaults to "http".
* `:method` determines whether events will be sent using GET or POST. It defaults to "get".
* `:port` determines the port to use. If you wish to set events over HTTPS, you should usually set it to 443.
* `:buffer_size` is the number of events which will be buffered before they are all sent simultaneously. The process of sending all buffered events is called "flushing". When using GET, `buffer_size` defaults to `0` because each request can only contain one event. When using POST, `buffer_size` defaults to `10`, and the buffered events are all sent together in a single request.
* `:on_success` is a callback which is called every time the buffer is flushed and every event in it is sent successfully (meaning with status code 200). It should accept one argument: the number of requests sent this way.
* `on_failure` is a callback which is called if the buffer is flushed but not every event is sent successfully. It should accept two arguments: the number of successfully sent events and an array containing the unsuccessful events.

<a name="async-emitter" />
### 5.2. The AsyncEmitter class

AsyncEmitter is a subclass of Emitter. It's API is exactly the same. It's advantage is that it always creates a new thread to flush its buffer, so requests are sent asynchronously.

**A note on testing:** if you test the AsyncEmitter by using a short script to send an event, you may find that the event fails to send. This is because the process exits before the flushing thread is finished. You can get round this either by adding a `sleep(10)` to the end of your script, or by using the [synchronous flush](#flushing).

<a name="multiple-emitters" />
### 5.3. Multiple emitters

It is possible to initialize a tracker with an array of emitters, in which case events will be sent to all of them:

```ruby
# Create a tracker with multiple emitters
my_tracker = SnowplowTracker::Tracker.new([my_sync_emitter, my_async_emitter], 'my_tracker_name', 'my_app_id')
```

You can also add new emitters after creating a tracker with the `add_emitter` method:

```ruby
# Create a tracker with multiple emitters
my_tracker.add_emitter(another_emitter)
```

<a name="flushing" />
### 5.4. Flushing manually

You may want to force an emitter to send all events in its buffer, even if the buffer is not full. The `Tracker` class has a `flush` method which flushes all its emitters. It accepts one argument, `sync`, which defaults to false. If you set `sync` to `true`, the flush will be synchronous: it will block until all flushing threads are finished.

```ruby
# Asynchronous flush
my_tracker.flush

# Synchronous flush
my_tracker.flush(true)
```

<a name="contracts" />
### 6. Contracts

The Snowplow Ruby Tracker uses the [Ruby Contracts gem][contracts] for typechecking. Contracts are enabled by default but can be turned on or off:

```ruby
# Turn contracts off
SnowplowTracker::disable_contracts

# Turn contracts back on
SnowplowTracker::enable_contracts
```

<a name="logging" />
## 7. Logging

The emitters.rb module has Ruby logging enabled to give you information about requests being sent. The logger prints messages about what emitters are doing. By default, only messages with priority "INFO" or higher will be logged.

To change this:

```ruby
require 'logger'
SnowplowTracker::LOGGER.level = Logger::DEBUG
```

The levels are:

| **Level**      | **Description** |
|---------------:|:----------------|
| `FATAL`  | Nothing logged     |
| `WARN`   | Notification for requests with status code not equal to 200  |
| `INFO`   | Notification for all requests |
| `DEBUG`  | Contents of all requests     |

[Back to top](#top)

<a name="advanced" />
## 8. Advanced usage

This section covers more advanced techniques with the Snowplow Ruby Tracker.

<a name="snowplow-ruby-duid" />
### 8.1. snowplow_ruby_duid

[snowplow_ruby_duid](https://github.com/simplybusiness/snowplow_ruby_duid/) is a Ruby gem that allows you to populate Snowplow's `domain_userid` cookie server-side from any Rack-based framework. This is useful if you want to fire an event on the user's initial request with the `domain_userid` already populated.

[Back to top](#top)

[ruby-0.2]: https://github.com/snowplow/snowplow/wiki/Ruby-Tracker-v0.2
[ruby-0.3]: https://github.com/snowplow/snowplow/wiki/Ruby-Tracker-v0.3
[ruby-0.5]: https://github.com/snowplow/snowplow/wiki/Ruby-Tracker-v0.5
[ruby-latest]: https://github.com/snowplow/snowplow/wiki/Ruby-Tracker

[contexts]: http://snowplowanalytics.com/blog/2014/01/27/snowplow-javascript-tracker-0.13.0-released-with-custom-contexts/#contexts
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
[contracts]: https://github.com/egonSchiele/contracts.ruby

[snowplow-0.9.14]: https://github.com/snowplow/snowplow/releases/tag/0.9.14
[issue-1220]: https://github.com/snowplow/snowplow/issues/1220
[issue-1231]: https://github.com/snowplow/snowplow/issues/1231
