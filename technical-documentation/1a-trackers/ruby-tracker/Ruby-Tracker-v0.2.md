<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow technical documentation) > [**Trackers**](trackers) > Ruby Tracker

*This page refers to version 0.2.0 of the Snowplow Python Tracker. Documentation for the latest version, 0.3.0, is available:*

*[Version 0.3][ruby-0.3]*
*[Version 0.4][ruby-0.4]*
*[Version 0.5][ruby-0.5]*
*[Version 0.6][ruby-latest]*

## Contents

- 1. [Overview](#overview)  
- 2. [Initialization](#init)  
  - 2.1 [Requiring the module](#requiring)
  - 2.2 [Creating a tracker](#create-tracker)  
- 3 [Adding extra data](#add-data)
  - 3.1 [`set_platform`](#set-platform)
  - 3.2 [`set_user_id`](#set-user-id)
  - 3.3 [`set_screen_resolution`](#set-screen-resolution)
  - 3.4 [`set_viewport`](#set-viewport)
  - 3.5 [`set_color_depth`](#set-color-depth)
  - 3.6 [`set_timezone`](#set-timezone)
  - 3.7 [`set_lang`](#set-language)
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


<a name="overview" />
## 1. Overview

The Snowplow Ruby Tracker allows you to track Snowplow events in your Ruby applications and gems and Ruby on Rails web applications.

The tracker should be straightforward to use if you are comfortable with Ruby development; any prior experience with Snowplow"s [[Python Tracker]], [[JavaScript Tracker]], [[Lua Tracker]], Google Analytics or Mixpanel (which have similar APIs to Snowplow) is helpful but not necessary.

The Ruby Tracker and Python Tracker have very similiar functionality and APIs.

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
tracker = SnowplowTracker::Tracker.new('d3rkrsqld9gmqf.cloudfront.net')
```

This tracker will log events to http://d3rkrsqld9gmqf.cloudfront.net/i. There are three other optional parameters:

```ruby
def initialize(endpoint, namespace=nil, app_id=nil, encode_base64=true)
```

`namespace` is a name for the tracker which will be added to every event the tracker fires. This is useful if you have initialized more than one tracker. `app_id` is the unique ID for the Ruby application. `encode_base64` determines whether JSONs in the querystring for an event will be base64-encoded.

So a more complete tracker initialization example might look like this:

```ruby
tracker = SnowplowTracker::Tracker.new('d3rkrsqld9gmqf.cloudfront.net', 'cf', 'ID-ap00035', false)
```

[Back to top](#top)

<a name="add-data" />
## 3. Adding extra data

You can configure the a tracker instance with additional information about your application's environment or current user. This data will be attached to every event the tracker fires. Here are the available methods:

| **Function**                                      | **Description**                                        |
|--------------------------------------------------:|:-------------------------------------------------------|
| [`set_platform`](#set-platform)                   | Set the application platform     |
| [`set_user_id`](#set-user-id)                     | Set the user ID                   |
| [`set_screen_resolution`](#set-screen-resolution) | Set the screen resolution          |
| [`set_viewport`](#set-viewport)                   | Set the viewport dimensions  | 
| [`set_color_depth`](#set-color-depth)             | Set the screen color depth |
| [`set_timezone`](#set-timezone)                   | Set the timezone               |
| [`set_lang`](#set-language)                       | Set the language             |

<a name="set-platform" />
### 3.1 Set the tracker's platform with `set_platform`

The platform can be any on of `'pc'`, `'tv'`, `'mob'`, `'cnsl'`, or `'iot'`. The default platform is `'pc'`.

tracker.set_platform('mob')

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
### 3.4 Set the viewport dimensions with `set_screen_resolution`

Similarly, you can pass the viewport dimensions in to Snowplow. Again, both numbers should be positive integers and the order is width followed by height. Example:

```ruby
tracker.set_screen_resolution(300, 200)
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

<a name="validation" />
#### 4.1.1 Argument validation

Each `track_XXX` method expects arguments of a certain type. The types are validated using the Ruby Contracts library. If a check fails, a runtime error is thrown. The section for each `track_XXX` method specifies the expected argument types for that method.

<a name="context-arg" />
#### 4.1.2 Optional context argument

Each `track_XXX` method has `context` as its penultimate optional parameter. This is for an optional array of [self-describing custom context JSONs][self-describing-jsons] attached to the event. Each element of the `context` argument should be a hash whose keys are "schema", containing a pointer to the JSON schema against which the context will be validated, and "data", containing the context data itself. The "data" field should contain a flat hash of key-value pairs. 

**Important:** Even if only one custom context is being attached to an event, it still needs to be wrapped in an array.

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

[ruby-0.3]: https://github.com/snowplow/snowplow/wiki/Ruby-Tracker-v0.3
[ruby-0.4]: https://github.com/snowplow/snowplow/wiki/Ruby-Tracker-v0.4
[ruby-0.5]: https://github.com/snowplow/snowplow/wiki/Ruby-Tracker-v0.5
[ruby-latest]: https://github.com/snowplow/snowplow/wiki/Ruby-Tracker

[contexts]: http://snowplowanalytics.com/blog/2014/01/27/snowplow-javascript-tracker-0.13.0-released-with-custom-contexts/#contexts
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
