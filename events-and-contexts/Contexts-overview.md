[**HOME**](Home) » [**EVENTS AND CONTEXTS**](Events-and-Contexts) » Contexts Overview

## Overview

When an **event** occurs, it generally involves a number of *entities*, and takes place in a particular setting. For example, the search event we used in our [example event dictionary entry](Event-dictionary) might have the following entities associated with it:

1. A user entity, who performed the search
2. A web page in which the event occurred
3. A variation on the web page that was part of an A/B test
4. A set of e.g. products that were returned from the search

All the above are examples of "contexts".

> A **context** is the group of entities associated with or describing the setting in which an event has taken place.

What makes contexts interesting is that they are common across multiple different event types. For example, the following events for a retailer will all involve a "product" context:

- View product
- Select product
- Like product
- Add product to basket
- Purchase product
- Review product
- Recommend product

Our retailer might want to describe product using a number of fields including:

- SKU
- Name
- Unit price
- Category
- Tags

Rather than define all the set of product-related fields for all the different product-related events in their respective schemas, Snowplow makes it possible to define a single product schema, and pass this as a context with any product related event. Our product schema might look as follows:

```json
{
	"$schema": "http://iglucentral.com/schemas/com.snowplowanalytics.self-desc/schema/jsonschema/1-0-0#",
	"description": "Schema for a product context",
	"self": {
		"vendor": "com.acme_company",
		"name": "product",
		"format": "jsonschema",
		"version": "1-0-0"
	},
	"type": "object",
	"properties": {
		"sku": {
			"type": "string",
			"maxLength": 255
		},
		"name": {
			"type": "string",
			"maxLength": 1024
		},
		"unitPrice": {
			"type": "number"
		},
		"category": {
			"type": "string",
			"maxLength": 255
		},
		"tags": {
			"type": "string",
			"maxLength": 1024
		}
	},
	"minProperties":1,
	"additionalProperties": false
}
```

We can then track an add to basket event as follows, we can pass in a handful of fields that are specific to the add to basket event (e.g. the quantity of the product added), and pass the whole product object as a context.

Contexts provide a convenient way in Snowplow to schema common entities once, and then use those schemas across all the different events where those entities are relevant.

Moreover, *multiple* contexts of either different or the same type can be sent with a single event. For example, you could send multiple product impressions (one for each product) with a page view event (say, if a user has loaded a catalogue page) or a transaction event (if a user has multiple line items in a transaction).

We distinguish 2 types of contexts:

- [Predefined contexts](#predefined-contexts)
- [Custom contexts](#custom-contexts)

A contexts can be added to any event (function) as a last parameter (argument).

<a name="predefined-contexts" />

## Predefined contexts

Predefined contexts are Snowplow authored web contexts and are available with JavaScript tracker. They are enabled at the tracker initialization step and thus the associated data will be added automatically to any Snowplow event fired on the page.

Below is an example of the usage.

```javascript
snowplow_name_here("newTracker", "cf", "d3rkrsqld9gmqf.cloudfront.net", {
  appId: "cfe23a"
  },
  contexts: {
    webPage: true,
    performanceTiming: true,
    gaCookies: true,
    geolocation: false
  }
});
```

See [Javascript tracker](1-General-parameters-for-the-Javascript-tracker#2214-adding-predefined-contexts) for the specific parameters to be used with predefined contexts.

<a name="custom-contexts" />

## Custom contexts

Custom contexts let you add additional information about the circumstances surrounding an event in the form of a dictionary. More specifically, a dictionary is represented with a [self-describing JSON](http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/).

> [**Self-describing JSON**](https://github.com/snowplow/iglu/wiki/Self-describing-JSONs) is a standardized [JSON](http://www.json.org/) format which co-locates a reference to the instance's [JSON Schema](http://json-schema.org/) alongside the instance's data

The `contexts` argument to any method is always *optional*. If set, it must be a self-describing JSON including at least one `name: property` pair in JSON provided as a value to `data` property of the self-describing JSON, where `data` is the name for an individual context entry.

It generally looks like the one below:

```json
{
    "schema": "iglu:com.snowplowanalytics/ad_click/jsonschema/1-0-0",
    "data": {
        "bannerId": "4732ce23d345"
    }
}
```

A few dos and don'ts for *context names*:

- Do name each context entry however you like
- Do use a context name to identify a set of associated data points which make sense to your business
- Do use the same contexts across multiple different events and event types
- Don't use multiple different context names to refer to the same set of data points

A few dos and don'ts for the *JSON*s inside each context entry JSONs:

- Do use any of the data types supported by custom unstructured events
- Do use Snowplow datatype suffixes if the data type would otherwise be unclear
- Don't nest properties as with custom unstructured events, the structure must be flat

» Read more about [custom contexts](Custom-contexts)

## Further reading

To find out more about the concepts mentioned above and ultimately how to set up custom contexts and send them to Snowplow pipeline, follow the links below.

- [Custom contexts](Custom-contexts)
- [Self-describing JSON](http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/)
- [Events](Events-overview)
- [Event dictionary](Event-dictionary)
- [Schema registry](Schema-registry)
- [Iglu schema registry](Iglu-registry)
