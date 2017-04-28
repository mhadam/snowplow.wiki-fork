[**HOME**](Home) » [**EVENTS AND CONTEXTS**](Events-and-Contexts) » Event Dictionary

## Overview

### What is an event dictionary?

When we setup Snowplow, we need to make sure that we track all the events that are meaningful to our business, so that the data associated with those events is available in Snowplow for analysis.

When we come to analyse Snowplow data, we need to be able to look at the event data and understand, in an unambiguous way, what that data actually means i.e. what it represents.

An event dictionary is a crucial tool in both cases. It is a document that defines the universe of events that a company is interested in tracking. For each event, it defines:

1. What the event is. Often this might be illustrated e.g. with screenshots
2. What data is captured when the event occurs, that represents the event. This is a data schema for the event
3. Details on how the relevant Snowplow tracker has been setup to pass the event data into Snowplow

### What is the purpose of an event dictionary?

Event dictionaries serve three purposes:

1. They aid analysis, by making sure that everyone using the data understands what each line of data "means". This is especially important as companies get larger, and analysts need to crunch data that was defined and instrumented prior to the analyst joining the company.
2. They aid technical setup: instrumentation (tracker setup) is driven by the event dictionary.
3. They can assist both the product management and analytics development process, by ensuring that the analytics instrumentation "keeps up" with an evolving product.

### Examples

Before considering an example you might wish to brush up on your understanding of what [JSON](http://www.json.org/) and [JSON schema](http://json-schema.org/) are. Also we have introduced a [self-describing JSON](https://github.com/snowplow/iglu/wiki/Self-describing-JSONs) and as a result a [self-describing JSON schema](https://github.com/snowplow/iglu/wiki/Self-describing-JSON-Schemas).

> [**Self-describing JSON**](https://github.com/snowplow/iglu/wiki/Self-describing-JSONs) - a standardized [JSON](http://www.json.org/) format which co-locates a reference to the instance's [JSON Schema](http://json-schema.org/) alongside the instance's data

An example entry in an event dictionary might look like this:

**Event**: ***perform_search***

*Screenshot*:

![screenshot](http://snowplowanalytics.com/assets/img/analytics/basic-concepts/perform-search-mockup.png)

*Event schema*:

```json
{
    "$schema": "http://iglucentral.com/schemas/com.snowplowanalytics.self-desc/schema/jsonschema/1-0-0#",
    "description": "Schema for performing a search",
    "self": {
        "vendor": "com.acme_company",
        "name": "perform_search",
        "format": "jsonschema",
        "version": "1-0-0"
    },

    "type": "object",
    "properties": {
    	"query": {
            "type": "string",
            "maxLength": 1024
        }
    },
    "minProperties":1,
    "additionalProperties": false
}
```

*Example data*:

```javascript
window.snowplow('trackUnstructEvent', {
    schema: 'iglu:com.acme_company/perform_search/jsonschema/1-0-0',
    data: {
        query: 'Bruce Springsteen DVD'
    }
});
```

When it comes to actual implementation of *event dictionaries* we prefer to call it a **schema registry**.

» Read more about building your own [schema registry](Schema-registry)

## Further reading

Snowplow privides a number of predefined (most commonly used) events such as page views, link clicks, form submissions and so on. We built the corresponding schema registries (event dictionaries). Now we need to have some kind of storage to keep the JSON schemas in and be able to retrieve them when the data is validated and processed to ensure the data conforms to the requirements set in those JSON schemas. Thus, we come up with the idea of JSON schema repository which we called **Iglu**.

Likewise once you produced JSON schemas for all the *custom* unstructured events (also called *self-describing events*) and contexts you would have to have your own JSON schema repository hosting your own JSON schemas.

To find out more about the concepts mentioned above and ultimately how to set up your own Iglu repository for Snowplow to be able to handle custom unstructured events and contexts, follow the links below.

- [Schema registry](Schema-registry)
- [Custom events](Custom-events)
- [Custom contexts](Custom-contexts)
- [Iglu schema registry](Iglu-registry)
