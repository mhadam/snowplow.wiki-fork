[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow technical documentation) > [**Storage**](storage documentation) > Kinesis Elasticsearch Sink

The Kinesis Elasticsearch Sink converts enriched Snowplow events from a Kinesis stream to JSON and writes them to an Elasticsearch cluster.

The resulting JSONs are like the [Snowplow Canonical Event](canonical-event-model) model with the following changes:

### Boolean fields reformatted

All boolean fields like `br_features_java` are either `"0"` or `"1"` in the canonical event model. The JSON converts these values to `false` and `true`.

### New `geo_location` field

The `geo_latitude` and `geo_longitude` fields are combined into a single `geo_location` field of Elasticsearch's ["geo_point" type][geopoint].

### Unstructured events

Unstructured events are expanded into full JSONs. For example, the event

```
{
    "schema": "iglu:com.snowplowanalytics.snowplow/link_click/jsonschema/1-0-1",
    "data": {
        "targetUrl": "http://snowplowanalytics.com/analytics/index.html",
        "elementId": "action",
        "elementClasses": [],
        "elementTarget": ""
	}
}
```

would be converted to the field

```
{
    "unstruct_com_snowplowanalytics_snowplow_link_click_1": {
        "targetUrl": "http://snowplowanalytics.com/analytics/index.html",
        "elementId": "action",
        "elementClasses": [],
        "elementTarget": ""
    }
}
```

### Custom contexts

Each custom context in an array is similarly expanded to a JSON with its own field. For example, the array

```
[
    {
        "schema": "iglu:com.acme/contextOne/jsonschema/1-0-0",
        "data": {
            "key": "value"
        }
    }
    {
        "schema": "iglu:com.acme/contextTwo/jsonschema/3-0-0",
        "data": {
            "name": "second"
        }
    }
]
```

would be converted to

```
{
    "contexts_com_acme_context_one_1": {
        "key": "value"
    },
    "contexts_com_acme_context_two_3": {
        "name": "second"
    }    
}
```

See also the [setup guide][setup-guide].

[geopoint]: http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/mapping-geo-point-type.html
[setup-guide]: Kinesis-Elasticsearch-Sink-Setup
