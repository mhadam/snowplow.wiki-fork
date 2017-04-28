[**HOME**](Home) » [**EVENTS AND CONTEXTS**](Events-and-Contexts) » Events Overview

## Overview

### What is an event?

An **event** is something that occurred in a particular point in time. Examples of events include:

- Load a web page
- Add an item to basket
- Enter a destination
- Check a balance
- Search for an item
- Share a video

**Snowplow** is an *event analytics platform*. Once you have set up one or more [Snowplow trackers](Setting-up-a-tracker), every time an event occurs, Snowplow should spot the event, generate a packet of data to describe the event, and send that event into the *Snowplow data pipeline*.


> The **Snowplow pipeline** is built to enable a very clean separation of the following steps in the data processing flow:
>
> - [Data collection](setting-up-a-collector)
> - [Data enrichment](setting-up-enrich)
> - [Data modelling](getting-started-with-data-modeling)
> - [Data analysis](getting-started-analyzing-snowplow-data)

To use Snowplow successfully, you need to have a good idea of:

- What events you care about in your business
- What events occur in your website / mobile application / server side systems / factories / call centers / dispatch centers
- What decisions you make based on those events
- What you need to know about those events to make those decisions

## Snowplow authored events

Snowplow supports a large number of events "out of the box" (first class citizens), most of which are fairly standard in a web analytics context. Examples of events that we support include:

- Page views
- Page pings
- Link clicks
- Form fill-ins (for the web)
- Form submissions
- Transactions

For events that Snowplow natively supports, there is generally a specific API for tracking that event type in Snowplow.

In general, each tracker will have a specific API call for tracking any events that have been defined by the Snowplow team, and you should refer to the [tracker-specific documentation](trackers) to make sure that this is set up correctly.

» Read more about [Snowplow authored events](Snowplow-authored-events)

## Custom events

If you wish to track an event that Snowplow does not recognise as a first class citizen (i.e. one of the events listed above), then you can track them using either the generic *custom structured event* or *self-describing events* (also called *unstructured events*).

When you track a **structured event**, you get five parameters:

- *Category*: The name for the group of objects you want to track.
- *Action*: A string that is used to define the user in action for the category of object.
- *Label*: An optional string which identifies the specific object being actioned.
- *Property*: An optional string describing the object or the action performed on it.
- *Value*: An optional numeric data to quantify or further describe the user action.

You may wish to track events on your website or application which are not directly supported by Snowplow and which *structured* event tracking does not adequately capture. Your event may have more than the five fields offered by `trackStructEvent`, or its fields may not fit into the category-action-label-property-value model. The solution is Snowplow's **self-describing events** (also called *unstructured events*). Self-describing events use *self-describing JSON* which can have arbitrarily many fields.

» Read more about [custom events](Custom-events)

## Further reading

To find out more about the concepts mentioned above and ultimately how to set up custom events and send them to Snowplow pipeline, follow the links below.

- [Snowplow authored events](Snowplow-authored-events)
- [Custom events](Custom-events)
- [Contexts overview](Contexts-overview)
- [Event dictionary](Event-dictionary)
- [Schema registry](Schema-registry)
- [Iglu schema registry](Iglu-registry)
