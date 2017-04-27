[**HOME**](Home) » [**EVENTS AND CONTEXTS**](Events-and-Contexts) » [Events Overview](Events-overview) » Snowplow Authored Events

Snowplow supports a large number of events "out of the box" (first class citizens), most of which are fairly standard in a web analytics context. Examples of events that we support include:

- Page views
- Page pings
- Link clicks
- Form fill-ins (for the web)
- Form submissions
- Transactions

For events that Snowplow natively supports, there is generally a specific API for tracking that event type in Snowplow. For example, if you want to track a page view using the Javascript tracker, you do so with the following Javascript:

```javascript
window.snowplow('trackPageView');
```

Whereas if you were tracking a pageview in an iOS app using the objective-c tracker, you’d do so like this:

```
[t1 trackPageView:@"www.example.com" title:@"example" referrer:@"www.referrer.com"];
```

In general, each tracker will have a specific API call for tracking any events that have been defined by the Snowplow team, and you should refer to the [tracker-specific documentation](trackers) to make sure that this is set up correctly.

### Further reading

To find out more about the events and ultimately how to set up custom events and contexts and send them to Snowplow pipeline, follow the links below.

- [Custom events](Custom-events)
- [Custom contexts](Custom-contexts)
- [Event dictionary](Event-dictionary)
- [Schema registry](Schema-registry)
- [Iglu schema registry](Iglu-registry)
