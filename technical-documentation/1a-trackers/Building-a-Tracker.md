[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > [**Trackers**](trackers) > Building a Tracker

## Building a Tracker

### Introduction

This guide is intended to help build a Snowplow Tracker from scratch. The best source of guidance is the source code for existing trackers, all of which are available on GitHub. But several of those trackers are in a fairly advanced state, so it may helpful to get an explanation of how to get started. (It is also worth checking out the early versions of those trackers, and looking at their GitHub issues labelled "Bug" to see common pitfalls.)

The [[Tracker Specification]] provides a list of possible tracker features.

### Create a Payload class

This will be a very simple class which maintains a map of name-value pairs. The names will be the names of the fields in the Snowplow Tracker Protocol; the values will be the data for those fields.

As well as being able to simply add single name-value pair to the map, it should also be possible to add a JSON object by first converting it to a string (and then base 64 encoding it if base 64 encoding is turned on).

### Create an Emitter class

#### Getting started: GET support

The Emitter class will be responsible for sending the events to a Snowplow collector.

This class should be configured with an HTTP endpoint and should have an `input` method which takes a Payload containing a map of key-value pairs and sends it to the endpoint in the form of a querystring, e.g.

```
e = new Emitter("mycollector.com")

p = Payload containing the map {
	"e": "se",
	"se_ca": "clothing",
	"se_ac": "add_to_basket"
}

e.input(p)
```

would send a GET request to `mycollector.com/i?e=se&se_ca=clothing&se_ac=add_to_basket`.

Make sure that the querystring is percent-encoded (http://en.wikipedia.org/wiki/Percent-encoding).

More advanced emitters will have the ability to send events asynchronously.

#### Adding POST support

Adding POST support should be done after completing the rest of this guide.

A single POST request can contain multiple events. This is especially useful for mobile trackers, which try to minimize the total number of requests sent.

The "/i" collector path is specific to GET requests - when the tracker comes to support POST requests, they will be sent to the path "/com.snowplowanalytics.snowplow/tp2", and the key-value pairs will be a JSON in the body of the request rather than the querystring.

POST requests should have a content type header of `application/json; charset=utf-8`.

The body of the POST request should be a [self-describing JSON](http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/) looking something like this:

```
{
	"schema": "iglu:com.snowplowanalytics.snowplow/payload_data/jsonschema/1-0-2",
	"data": [
		{
			"e": "se",
			...
		},
		{
			"e": "ue",
			...
		},
		...
	]
}
```

The inner `data` field is an array of JSONs, each corresponding to a single individual event.


### Create a Tracker class

This class will contain API methods for the various types of Snowplow event (`trackStructEvent`, `trackUnstructEvent`, `trackPageView`, and so on). These methods don't change much between implementations in different languages so the existing trackers and the [Snowplow Tracker Protocol](https://github.com/snowplow/snowplow/wiki/snowplow-tracker-protocol) should be a good guide.

The purpose of each of these methods is to take arguments provided by the user and assemble them into a map (using the Payload class) which then gets passed to the Emitter's `input` function to be sent to the collector.

Unless the user specifies the timestamp for an event, a timestamp should be attached automatically. A [v4 UUID](http://en.wikipedia.org/wiki/Universally_unique_identifier#Version_4_.28random.29) should also be attached to each event to help with deduplication.

As well as these methods for sending individual events, it should be possible to configure the tracker with persistent data which gets sent with every event - like the platform and the user ID and the user's timezone. (In more advanced Snowplow Trackers, there is a Subject class representing a single user which can be configured with these sorts of properties.)

In addition, the tracker should be initialized with a user-configured application ID and "tracker namespace" (so that if a user has set up two instances of the tracker, they can distinguish which events came from which tracker by giving the instances different tracker namespaces.) These should be attached to every event, together with the version of the tracker.

### Unstructured events and custom contexts

Snowplow lets you send an array of custom contexts with each event, so each tracker method which fires an event should have an optional `contexts` argument for this array. The elements of this array should be [self-describing JSONs](http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/). Before being attached to the payload, these JSONs should be stringified and (if the user has enabled base 64 encoding) base 64 encoded. If encoding is turned on, the querystring parameter for the array of contexts is `co`; otherwise it is `cx`. This difference is so that the enrichment process knows whether to attempt to base 64 decode the field.

The `trackUnstructEvent` tracker method follows similar principles, but it takes a single self-describing JSON rather than an array. If encoding is turned on, the querystring parameter for the unstructured event JSON is `ue_pr`; otherwise it is `ue_px`.

### Testing

You can use the RequestBin website to stand in for the Snowplow collector so you can inspect the querystrings generated by your tracker and check that they are formed as expected. You can also use Python's SimpleHTTPServer module from the command line, and use your tracker to send events to localhost on (for example) port 8080:

```
python -m SimpleHTTPServer 8080
```

The best way to make sure that the Tracker's output is acceptable to Snowplow's enrichment step is to actually put it through the enrichment process and check that no error results. You can do this using Snowplow's Kinesis pipeline.

This pipeline exists to process events in real time using Amazon Kinesis. But it can also be run in local mode, so that you can send events through the pipeline and check if they are valid without having to set up Kinesis.

To do this, download the Kinesis resources zipfile from the Snowplow Hosted Assets page. From it you will need the Scala Stream Collector and Stream Enrich. The former will accept the HTTP requests generated by your tracker, serialize them, and send them to the latter for enrichment. (An example of an enrichment would be converting currencies in a transaction event to a base currency.)

The wiki pages for setting up these two apps contain instructions for running them in local mode:

https://github.com/snowplow/snowplow/wiki/Configure-the-Scala-Stream-Collector
https://github.com/snowplow/snowplow/wiki/Configure-Stream-Enrich

The command you use will look something like this:

```
./snowplow-stream-collector-0.x.0 --config collector-config.conf | ./snowplow-stream-enrich-0.x.0 --config enrichment-config.conf --resolver file:resolver.json
```

where the "x"s stand in for the latest versions of the two apps.

You can now run your tracker sending events to localhost:8080 (assuming you configured the Scala Stream Collector to listen on port 8080). Successfully enriched events will be printed to stdout in TSV form; failed events will be logged to stderr with list of messages describing what was unexpected about the input. For example, if you make a typo so that the querystring your tracker generates for a structured event contains `e=seZ` rather than `e=se`, you will see a JSON like this:

```
{"line":"CwBkAAAACTEyNy4wLjAuMQoAyAAAAU1CI+FjCwDSAAAABVVURi04CwDcAAAAEHNzYy0wLjQuMC1zdGRvdXQLASwAAAALY3VybC83LjMyLjALAUAAAAACL2kLAUoAAAAFZT1zZVoPAV4LAAAAAwAAAAtBY2NlcHQ6ICovKgAAABRIb3N0OiBsb2NhbGhvc3Q6ODA4MAAAABdVc2VyLUFnZW50OiBjdXJsLzcuMzIuMAsBkAAAAAlsb2NhbGhvc3QLAZoAAAAkZTA1MzI0ZWItNzFiYi00YWNlLTk0YTQtZjRiZmQ2OTdkMWNjC3ppAAAAQWlnbHU6Y29tLnNub3dwbG93YW5hbHl0aWNzLnNub3dwbG93L0NvbGxlY3RvclBheWxvYWQvdGhyaWZ0LzEtMC0wAA==","errors":["Field [e]: [seZ] is not a recognised event code"]}
```

The "line" field is the event which caused the error. The Scala Stream Collector serializes all events using Apache Thrift and then base 64 encodes them before sending them to Stream Enrich - this serialization is why the "line" appears unreadable. But the "errors" field should make the problem obvious.
