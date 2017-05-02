[**HOME**](Home) » [**EVENTS AND CONTEXTS**](Events-and-Contexts) » [Contexts Overview](Contexts-overview) » Custom Contexts

## Custom Contexts

Custom contexts can be used to augment any standard Snowplow event type, including unstructured events, with additional data.

Custom contexts can be added as an extra argument to any of Snowplow's `track..()` methods and to `addItem` and `addTrans`.

Each custom context is a **self-describing JSON**.

> [**Self-describing JSON**](https://github.com/snowplow/iglu/wiki/Self-describing-JSONs) is a standardized [JSON](http://www.json.org/) format which co-locates a reference to the instance's [JSON Schema](http://json-schema.org/) alongside the instance's data

If you want to create your own custom context, you must create a [JSON schema](http://json-schema.org/) for it and upload it to an [Iglu schema registry](Iglu-registry). Since more than one can be attached to an event, the `context` argument (if it is provided at all) should be a *non-empty array of self-describing JSONs*.

**Important:** Even if only one custom context is being attached to an event, it still needs to be wrapped in an array.

Here are two examples of custom context JSONs. One describes a page, and the other describes a user on that page.

```
{
    schema: "iglu:com.example_company/page/jsonschema/1-2-1",
    data: {
        pageType: 'test',
        lastUpdated: new Date(2016,3,10)
    }
}
```

```
{
    schema: "iglu:com.example_company/user/jsonschema/2-0-0",
    data: {
      userType: 'tester',
    }
}
```

Below is a JavaScript tracker example how the above custom contexts could be attached to a page view event:

```javascript
window.snowplow_name_here(
    'trackPageView',	// Snowplow authored "page view" event
    null, 				// no custom title
    [					// array of custom contexts
        {
            schema: "iglu:com.example_company/page/jsonschema/1-2-1",
            data: {
                pageType: 'test',
                lastUpdated: new Date(2016,3,10)
            }
        },
        {
            schema: "iglu:com.example_company/user/jsonschema/2-0-0",
            data: {
                userType: 'tester',
            }
        }
    ]
);
```

In this case an empty string has been provided to the optional `customTitle` argument in order to reach the `context` argument, which has to be the last argument of the functions.

Knowing in advance what the expected structure and format of data should be as a necessity to be able to handle custom contexts brought about an idea of what we call **schema registry**.

» Read more about [schema registry](Schema-registry)

## Further reading

To find out more about the concepts mentioned above and ultimately how to set up custom events and contexts and send them to Snowplow pipeline, follow the links below.

- [Self-describing JSON](http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/)
- [Events](Events-overview)
- [Contexts](Contexts-overview)
- [Event dictionary](Event-dictionary)
- [Schema registry](Schema-registry)
- [Iglu schema registry](Iglu-registry)
