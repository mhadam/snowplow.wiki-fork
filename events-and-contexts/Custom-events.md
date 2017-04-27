[**HOME**](Home) » [**EVENTS AND CONTEXTS**](Events-and-Contexts) » [Events Overview](Events-overview) » Custom Events

## Overview

Snowplow supports a large number of events "out of the box" (first class citizens), most of which are fairly standard in a web analytics context. See [Snowplow authored events](Snowplow-authored-events) for more details.

If you wish to track an event that Snowplow does not recognise as a first class citizen then you can track them using one of the following generic events:

- [Custom structured event](#structured-event)
- [Self-describing event](#unstructured-event) (also called *unstructured* event)

<a name="structured-event" />

## Custom structured event

With a structured event, we follow Google five-variable tracking event structure. When you track a structured event, you get five parameters:

- *Category*: The name for the group of objects you want to track.
- *Action*: A string that is used to define the user in action for the category of object.
- *Label*: An optional string which identifies the specific object being actioned.
- *Property*: An optional string describing the object or the action performed on it.
- *Value*: An optional numeric data to quantify or further describe the user action.

For example, when tracking a custom structured event the specification for the `trackStructEvent` method (Javascript tracker) follows the pattern:

```
snowplow_name_here('trackStructEvent', 'category','action','label','property','value');
```

If, say, you wanted to send your marketing emails out and capture who and when opened the email you could use Pixel tracker by embedding the pixel into your email in the following manner

```
<img src="https://collector.snplow.com/i?uid=user%40gmail.com&e=se&se_ca=email&se_ac=open&se_la=My%20Campaign&se_pr=%7Bproduct%3A%22ABC%22%2Cdiscount%3A%2210%25%22%7D">
```

Once email is open and the tracking pixel is loaded the following `key:value` should be captured by the collector:

```
uid=user@gmail.com                   //user ID (email address)
e=se                                 //event = custom structured event
se_ca=email                          //event_category = email
se_ac=open                           //event_action = open
se_la=My Campaign                    //event_label = "My Campaign"
se_pr={product:"ABC",discount:"10%"} //event_property = 10% discount on product ABC
```

Note the user email address could have been populated with the emailing tool (unique for each recipient) while time of event is captured automatically by Snowplow.

See [Snowplow tracker protocol](snowplow-tracker-protocol#39-custom-structured-event-tracking) for the specific parameters to be used with structured events and examples of usage.

<a name="unstructured-event" />

## Self-describing event

Self-describing events (also known as custom unstructured events) are user events which do not fit one of the existing [Snowplow event types](Snowplow-authored-events) (page views, ecommerce transactions, etc.), and do not fit easily into our existing [custom structured event format](#structured-event). A **self-describing event** consists of two elements:

- A `name` of the unstructured event, e.g. "Game saved" or "returned-order". This is case-sensitive; spaces etc are allowed
- A set of `key: value` properties (also known as a hash, associative array or dictionary)

You might recognize what we call self-describing events (custom unstructured events) from other analytics tools including MixPanel, KISSmetrics and Keen.io, where they are the primary trackable event type.

Self-describing events are great for a couple of use cases:

- Where you want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow)
- Where you want to track events which have unpredictable or frequently changing properties

The set of `key: value` properties in self-describing events is represented with **self-describing JSON** which can have arbitrarily many fields.

> [**Self-describing JSON**](https://github.com/snowplow/iglu/wiki/Self-describing-JSONs) is a standardized [JSON](http://www.json.org/) format which co-locates a reference to the instance's [JSON Schema](http://json-schema.org/) alongside the instance's data

For example, to track a self-describing event with Javascript tracker, you make use of the `trackUnstructEvent` method with the pattern shown below:

```
snowplow_name_here('trackUnstructEvent', <<SELF-DESCRIBING EVENT JSON>>);
```

More specific example using JavaScript tracker:

```
window.snowplow_name_here('trackUnstructEvent', {
    schema: 'iglu:com.acme_company/viewed_product/jsonschema/2-0-0',
    data: {
        productId: 'ASO01043',
        category: 'Dresses',
        brand: 'ACME',
        returning: true,
        price: 49.95,
        sizes: ['xs', 's', 'l', 'xl', 'xxl'],
        availableSince: new Date(2013,3,7)
    }
});
```

See [Snowplow tracker protocol](snowplow-tracker-protocol#310-custom-unstructured-event-tracking) for the specific parameters to be used with self-describing events and examples of usage.

Note that with **structured** event, we know the structure of the data and the value format in advance as it is predefined and has to follow the five-value structure. With **unstructured**  (self-describing) events, however, the number of `key:value` pairs can vary and is determined by business model of the entity associated with the event.

Therefore, for Snowplow to be able to validate and extract the data self-describing JSON would have to be sent with the event. By its definition, self-describing JSON includes a reference to JSON schema which has to be in place by the time enrichment process takes place. It allows for maximum customisation of the unstructured events.

Knowing in advance what the expected structure and format of data should be as a necessity to be able to handle events and contexts brought about an idea of what we call **schema registry**.

» Read more about [schema registry](Schema-registry)

## Further reading

To find out more about the concepts mentioned above and ultimately how to set up custom events and contexts and send them to Snowplow pipeline, follow the links below.

- [Self-describing JSON](http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/)
- [Custom structured events](Canonical-event-model#customstruct)
- [Unstructured events guide][unstructured-events]
- [Contexts](Contexts-overview)
- [Event dictionary](Event-dictionary)
- [Schema registry](Schema-registry)
- [Iglu schema registry](Iglu-registry)

[unstructured-events]: http://snowplowanalytics.com/blog/2013/05/14/snowplow-unstructured-events-guide/
