<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Trackers**](trackers) > [**JavaScript Tracker**](Javascript-Tracker) > General parameters

*This page refers to version 2.0.0 of the Snowplow JavaScript Tracker.*
*Click [here][general-parameters-v1] for the corresponding documentation for version 1.*
*Click [here][general-parameters] for the corresponding documentation for version 2.1.1.*

<a name="general" />

## 2. General parameters
  - 2.1 [Loading Snowplow.js](#loading)
  - 2.2 [Initialising a tracker](#initialisation)
    - 2.2.1 [Setting the application ID](#app-id)
    - 2.2.2 [Setting the platform](#platform)
    - 2.2.3 [Configuring the cookie domain](#cookie-domain)
    - 2.2.4 [Configuring the cookie name](#cookie-name)
    - 2.2.5 [Configuring base 64 encoding](#base-64)
    - 2.2.6 [Respecting Do Not Track](#respect-do-not-track)
    - 2.2.7 [User fingerprinting](#user-fingerprint)
    - 2.2.8 [Setting the user fingerprint seed](#user-fingerprint-seed)
    - 2.2.9 [Setting the page unload pause](#page-unload-timer)
    - 2.2.10 [Altering cookies](#write-cookies)
  - 2.3 [Other parameters](#other-methods)
    - 2.3.1 [Setting the user id](#user-id)
      - 2.3.1.1 [`setUserId`](#set-user-id)
      - 2.3.1.2 [`setUserIdFromLocation`](#set-user-id-from-location)
      - 2.3.1.3 [`setUserIdFromReferrer`](#set-user-id-from-referrer)
      - 2.3.1.4 [`setUserIdFromCookie`](#set-user-id-from-cookie)
    - 2.3.2 [Setting a custom URL with `setCustomUrl`](#custom-url)
    - 2.3.3 [Configuring cookie timeouts using `setSessionCookieTimeout`](#cookie-timeout)
  - 2.4 [Managing multiple trackers](#multiple-trackers)
  - 2.5 [How the Tracker uses cookies](#cookies)

<a name="loading"/>

### 2.1 Loading Snowplow.js

Use the following tag to your page to load Snowplow.js:

```html
<script type="text/javascript" async=1>
;(function(p,l,o,w,i,n,g){if(!p[i]){p.GlobalSnowplowNamespace=p.GlobalSnowplowNamespace||[];
p.GlobalSnowplowNamespace.push(i);p[i]=function(){(p[i].q=p[i].q||[]).push(arguments)
};p[i].q=p[i].q||[];n=l.createElement(o);g=l.getElementsByTagName(o)[0];n.async=1;
n.src=w;g.parentNode.insertBefore(n,g)}}(window,document,"script","//d1fc8wv8zag5ca.cloudfront.net/2.0.0/sp.js","snowplow_name_here"));
</script>
```

*Important note regarding testing:* `"//d1fc8wv8zag5ca.cloudfront.net/2.0.0/sp.js"` is the protocol-relative URL used to fetch sp.js. It will work if the your web page is using the "http" or "https" protocol. But if you are testing locally and loading your page from your filesystem using the "file" protocol (so its URI looks something like "file:///home/joe/snowplow_test.html"), the protocol-relative URL will also use that protocol, preventing the script from loading. To avoid this, change the URL to `"http://d1fc8wv8zag5ca.cloudfront.net/2.0.0/sp.js"` when testing locally.

As well as loading Snowplow, this tag creates a global function called "snowplow_name_here" which you use to access the Tracker. You can replace the string "snowplow_name_here" with the function name of your choice. This is encouraged: if there are two Snowplow users on the same page, there won't be any conflict between them as long as they have chosen different function names. The rest of the documentation will assume that the function is called "snowplow_name_here".

Once the function `snowplow_name_here` is created, the syntax for using Snowplow methods is as follows:

```javascript
snowplow_name_here({{"methodName"}}, {{first method argument}}, {{second method argument}}, ...);
```

For example, the method `trackStructEvent` has this signature:

```javascript
function trackStructEvent(category, action, label, property, value, context)
```

where only the first two arguments are required. You would use it like this:

```javascript
snowplow_name_here('trackStructEvent', 'Mixes', 'Play', '', '', 20);
```

Empty strings are provided for the label and value arguments to pad them out. (`Null` would also work.) They won't be added to the event. Neither will the context argument, which isn't provided at all.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="initialisation" />

### 2.2 Initialising a tracker

Tracker initialisation takes three arguments:

1. The tracker name
2. The collector endpoint
3. An optional argmap containing other settings

Here is a simple example of how to initialise a tracker:

```javascript
snowplow_name_here("newTracker", "cf", "d3rkrsqld9gmqf.cloudfront.net", {
  appId: "cfe23a",
  platform: "mob"
});
```

The tracker will be named "cf" and will send events to d3rkrsqld9gmqf.cloudfront.net, the cloudfront collector specified. The final argument is called the argmap. Here it is just used to set the app ID and platform for the tracker. Each event the tracker sends will have an app ID field set to "cfe23a" and a platform field set to "mob".

Here is a longer example in which every tracker configuration parameter is set:

```javascript
snowplow_name_here("newTracker", "cf", "d3rkrsqld9gmqf.cloudfront.net", {
  appId: "cfe23a",
  platform: "mob"
  cookieDomain: null,
  cookieName: "_sp534_",
  encodeBase64: false,
  respectDoNotTrack: false,
  userFingerprint: true,
  userFingerprintSeed: 6385926734,
  pageUnloadTimer: 0,
  writeCookies: true
});
```

We will now go through the various argmap parameters. Note that these are all optional. In fact, you aren't required to provide any argmap at all.

<a name="app-id" />

#### 2.2.1 Setting the application ID

Set the application ID using the `appId` field of the argmap. This will be attached to every event the tracker fires. You can set different application IDs on different parts of your site. You can then distinguish events that occur on different applications by grouping results based on `application_id`.

<a name="platform" />

#### 2.2.2 Setting the platform
Set the application platform using the `platform` field of the argmap. This will be attached to every event the tracker fires. Its default value is "web". For a list of supported platforms, please see the [Snowplow Tracker Protocol][snowplow-tracker-protocol].

<a name="cookie-domain" />

#### 2.2.3 Configuring the cookie domain

If your website spans multiple subdomains e.g.

* www.mysite.com
* blog.mysite.com
* application.mysite.com

You will want to track user behaviour across all those subdomains, rather than within each individually. As a result, it is important that the domain for your first party cookies is set to '.mysite.com' rather than 'www.mysite.com'. By doing so, any values that are stored on the cookie on one of subdomain will be accessible on all the others.

Set the cookie domain for the tracker instance using the `cookieDomain` field of the argmap. If this field is not set, the cookies will not be given a domain.

<a name="cookie-name" />

#### 2.2.4 Configuring the cookie name

Set the cookie name for the tracker instance using the `cookieName` field of the argmap. The default is "_sp_". Snowplow uses two cookies, a domain cookie and a session cookie. In the default case, their names are "_sp_id" and "_sp_ses" respectively. If you are upgrading from an earlier version of Snowplow, you should use the default cookie name so that the cookies set by the earlier version are still remembered. Otherwise you should provide a new name to prevent clashes with other Snowplow users on the same page.

<a name="base-64">
#### 2.2.5 Configuring base 64 encoding
By default, unstructured events and custom contexts are encoded into Base64 to ensure that no data is lost or corrupted. You can turn encoding on or off using the `encodeBase64` field of the argmap.

<a name="respect-do-not-track" />

#### 2.2.6 Respecting Do Not Track

Most browsers have a Do Not Track option which allows users to express a preference not to be tracked. You can respect that preference by setting the `respectDoNotTrack` field of the argmap to `true`. This prevents cookies from being sent and events from being fired.

<a name="user-fingerprint" />

#### 2.2.7 User fingerprinting

By default, the tracker generates a user fingerprint based on various browser features. This fingerprint is likely to be unique and so can be used to track anonymous users. You can turn user fingerprinting off by setting the `userFingerprint` field of the argmap to `false`.

<a name="user-fingerprint-seed" />

#### 2.2.8 Setting the user fingerprint seed

The `userFingerprintSeed` field of the the argmap lets you choose the hash seed used to generate the user fingerprint. If this is not specified, the default is 123412414.

<a name="page-unload-timer" />

#### 2.2.9 Setting the page unload pause

Whenever the Snowplow Javascript Tracker fires an event, it automatically starts a 500 millisecond timer running. If the user clicks on a link or refreshes the page during this period (or, more likely, if the event was triggered by the user clicking a link), the page will wait until the timer is finished before unloading. 500 milliseconds is usually enough to ensure the event has time to be sent.

You can change the pause length (in milliseconds) using the `pageUnloadTimer` of the argmap. The above example completely eliminates the pause. This does make it unlikely that events triggered by link clicks will be sent.

<a name="write-cookies" />

#### 2.2.10 Altering cookies

The `writeCookies` argument is a boolean value which determines whether the tracker instance will be able to alter cookies or add new ones. It does not affect whether the tracker instance will read cookies, so if it is turned off but Snowplow cookies with the tracker's configured cookie name already exist for the page, the tracker will continue to report those cookies' values. If you do use this argument, be careful - if two trackers on the same page are both initialised with the same cookie name and with `writeCookies` turned on, inaccurate data will result from them both trying to alter the same cookies. Note that you will always be fine if the `writeCookies` argument is not set - because the default behaviour avoids these problems.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="other-methods" />

### 2.3 Other parameters

<a name="user-id" />

#### 2.3.1 Setting the user ID

The JavaScript Tracker automatically sets a `domain_userid` based on a first party cookie.

There are many situations, however, when you will want to identify a specific user using an ID generated by one of your business systems. To do this, you use one of the methods described in this section: `setUserId`, `setUserIdFromLocation`, `setUserIdFromReferrer`, and `setUserIdFromCookie`.

Typically, companies do this at points in the customer journey when the user identifies him / herself e.g. if he / she logs in.

Note: this will only set the user ID on further events fired while the user is on this page; if you want events on another page to record this user ID too, you must call `setUserId` on the other page as well.

<a name="set-user-id" />

##### 2.3.1.1 `setUserId`

`setUserId` is the simplest of the four methods. It sets the business user ID to a string of your choice:

```javascript
snowplow_name_here('setUserId', 'joe.blogs@email.com');
```

<a name="set-user-id-from-location" />

##### 2.3.1.1 `setUserIdFromLocation`

`setUserIdFromLocation` lets you set the user ID based on a querystring field of your choice. For example, if the URL is `http://www.mysite.com/home?id=user345`, then the following code would set the user ID to "user345":

```javascript
snowplow_name_here('setUserIdFromLocation', 'id');
```

<a name="set-user-id-from-referrer" />

##### 2.3.1.1 `setUserIdFromReferrer`

`setUserIdFromReferrer functions in the same way as `setUserIdFromLocation`, except that it uses the referrer querystring rather than the querystring of the current page.


```javascript
snowplow_name_here('setUserIdFromReferrer', 'id');
```

<a name="set-user-id-from-cookie" />

##### 2.3.1.1 `setUserIdFromCookie`

Use `setUserIdFromCookie` to set the value of a cookie as the user ID. For example, if you have a cookie called "cookieid" whose value is "user123", the following code would set the user ID to "user123":

```javascript
snowplow_name_here('setUserIdFromCookie', 'cookieid');
```

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="custom-url" />

#### 2.3.2 Setting a custom URL with `setCustomUrl`

The Snowplow JavaScript Tracker automatically tracks the page URL on any event tracked. However, in certain situations, you may want to override the actual URL with a custom value. (For example, this might be desirable if your CMS spits out particularly ugly URLs that are hard to unpick at analysis time.) In that case, you can override the default value using the `setCustomUrl` function.

To set a custom URL, use the `setCustomUrl` method:

```javascript
snowplow_name_here('setCustomUrl', 'http://mysite.com/checkout-page');
```

<a name="cookie-timeout" />

#### 2.3.3 Configuring cookie timeouts using `setSessionCookieTimeout`

The JavaScript Tracker sets two cookies: a visitor cookie and a session cookie. The visitor cookie contains all persistent information about the user, including a visit count (the number of times the user has visited the site). It lasts for two years. The session cookie is specific to an individual session. By default, it expires after 30 minutes pass with no event fired. Whenever a Snowplow event is fired, if no session cookie is found, the Tracker takes this to mean that a new session has started. It therefore increments the visitor cookie's visit count. If the user leaves the site and returns before the 30 minutes is up, the visit count is not incremented.

The visit count is added to each event querystring as "vid". "vid=3" would mean an event was fired during the user's third session.

You can change the default from 30 minutes by using `setSessionCookieTimeout`. You should give the expiration time of the session cookie in seconds:

```javascript
snowplow_name_here('setSessionCookieTimeout', 3600);
```

The above code would cause the session cookie to last for one hour.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="multiple-trackers" />

### 2.4 Managing multiple trackers

You have more than one tracker instance running on the same page at once. This may be useful if you want to log events to different collectors. By default, any Snowplow method you call will be executed by every tracker you have created so far:

```javascript
snowplow_name_here("newTracker", "cf1", "d3rkrsqld9gmqf.cloudfront.net", {
  appId: "cfe23a",
  platform: "mob"
});

snowplow_name_here("newTracker", "cf2", "a5grvrhue7ewvt.cloudfront.net", {
  appId: "cfe23a",
  platform: "mob"
});

// Both trackers will use this custom title
snowplow_name_here('setCustomUrl', 'http://mysite.com/checkout-page');

// Both trackers will fire a structured event
snowplow_name_here('trackStructEvent', 'Mixes', 'Play', 'MrC/fabric-0503-mix', '', '0.0');
```

You can override this behaviour and specify which tracker instances will execute a Snowplow method. To do this, change the method name by adding a colon followed by a list of tracker namespaces separated by semicolons. The namespace of a tracker instance is the first argument to the "newTracker" method. In our example, the tracker instances have namespaces "cf1" and "cf2".

```javascript
// Only the first tracker will fire this structured event
snowplow_name_here('trackStructEvent:cf1', 'Mixes', 'Play', 'MrC/fabric-0503-mix', '', '0.0');

// Only the second tracker will fire this unstructured event
snowplow_name_here('trackUnstructEvent:cf2', 'com.acme_company' 'Viewed Product',
    {
        product_id: 'ASO01043',
        category: 'Dresses',
        brand: 'ACME',
        returning: true,
        price: 49.95,
        sizes: ['xs', 's', 'l', 'xl', 'xxl'],
        available_since$dt: new Date(2013,3,7)
    }
);

// Both trackers will fire a page view event
snowplow_name_here('trackPageView:cf1;cf2');
```

<a name="cookies" />

### 2.5 How the Tracker uses cookies

Unless you have enabled `respectDoNotTrack` in the configuration argmap, the tracker will use cookies to persist information. There are two first party cookies: the session cookie and the ID cookie. By default their names are prefixed with "_sp_", but you can change this using the "cookieName" field in the argmap.

#### The session cookie

Called _sp_ses by default, the only purpose of this cookie is to differentiate between different visits. Whenever an event is fired, the session cookie is set to expire in 30 minutes. (This value can be altered using `setSessionCookieTimeout`.)

If no session cookie is already present when an event fires, the tracker treats this as an indication that long enough has passed since the user last visited that this session should be treated as a new session rather than a continuation of the previous session. The `visitCount` (how many times the user has visited) is increased by one and the `lastVisitTs` (the timestamp for the last session) is updated.

#### The ID cookie

This cookie is called _sp_id by default. It is used to persist information about a user's activity on the domain between sessions. It contains the following information:

* An ID for the user based on a hash of various browser attributes
* How many times the user has visited the domain
* The time of the current visit
* The time of the last visit

It expires after 2 years.

#### The Clojure Collector cookie

There is a third sort of Snowplow-related cookie: the cookie set by the [Clojure Collector][clojure-collector], independently of the JavaScript Tracker. If you are using another type of collector, this cookie will not be set. The Clojure Collector cookie is called "sp". It is a third-party cookie used to track users over multiple domains. It expires after one year.

[contents]: Javascript-Tracker
[general-parameters-v1]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker-v1
[general-parameters]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker
[snowplow-tracker-protocol]: https://github.com/snowplow/snowplow/wiki/SnowPlow-Tracker-Protocol
[contexts]: https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker-v1#custom-contexts
[clojure-collector]: https://github.com/snowplow/snowplow/wiki/Clojure-collector
