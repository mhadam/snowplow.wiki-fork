[**HOME**](Home) » [**EVENTS AND CONTEXTS**](Events-and-Contexts) » [Event Dictionary](Event-dictionary) » Iglu Schema Registry

## Overview

**Iglu** is a machine-readable, open-source schema registry for [JSON schema](http://json-schema.org/) and Thrift schema from the team at [Snowplow Analytics](http://snowplowanalytics.com/). A schema registry is like *npm* or *Maven* or *git* but holds data schemas instead of software or code.

Iglu consists of three key technical aspects:

1. A [common architecture](https://github.com/snowplow/iglu/wiki/Common-architecture) that informs all aspects of Iglu
2. [Iglu clients](https://github.com/snowplow/iglu/wiki/Iglu-clients) that can resolve schemas from one or more Iglu registries
3. [Iglu registries](https://github.com/snowplow/iglu/wiki/Iglu-repositories) that can host a set of [self-describing JSON schemas](http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/)

## Iglu explained

**Iglu** is built on a set of technical design decisions. It is this set of design decisions that allow Iglu clients and registries to interoperate. Please review the following design documents:

- [**Self-describing JSON schema**](https://github.com/snowplow/iglu/wiki/Self-describing-JSON-Schemas) - simple extensions to JSON schema which semantically identify and version a given JSON schema
- [**Self-describing JSON**](https://github.com/snowplow/iglu/wiki/Self-describing-JSONs) - a standardized JSON format which co-locates a reference to the instance's JSON schema alongside the instance's data
- [**SchemaVer**](https://github.com/snowplow/iglu/wiki/SchemaVer) - how we semantically version schemas -
*Schema resolution* - our public algorithm for how we determine in which order we check Iglu registries for a given schema

**Iglu clients** are used for interacting with Iglu server repos and for resolving schemas in embedded and remote Iglu schema registries.

In the below diagram we show an Iglu client resolving a schema from Iglu Central, one embedded registry and a further two remote HTTP registries:

![Iglu client](https://github.com/snowplow/iglu/wiki/technical-documentation/images/iglu-clients.png)

An **Iglu registry** acts as a store of data schemas. Hosting JSON schemas in an Iglu registry allows you to use those schemas in Iglu-capable systems such as [Snowplow](https://github.com/snowplow/snowplow/wiki/).

So far we support two types of Iglu registry:

- **Remote registries** - essentially websites containing schemas which an Iglu client can query over HTTP
- **Embedded registries** - which are embedded in a piece of software (typically alongside an Iglu client)

In the below diagram we show an Iglu client resolving a schema from Iglu Central, one embedded registry and a further two remote HTTP registries:

![Iglu repositories](https://github.com/snowplow/iglu/wiki/technical-documentation/images/iglu-repos.png)

**Iglu Central** (http://iglucentral.com) is a public registry of JSON schemas hosted by [Snowplow Analytics](http://snowplowanalytics.com/).

Under the covers, Iglu Central is built and run as a **static Iglu registry**, hosted on Amazon S3.

> A **static repo** is simply an Iglu registry server structured as a static website.

![Iglu Central](https://github.com/snowplow/iglu/wiki/technical-documentation/images/iglu-central.png)

The **deployment process** for Iglu Central is documented on [this wiki](https://github.com/snowplow/iglu/wiki/Iglu-Central-setup) in case a user wants to setup a public mirror or private instance of Iglu Central.

### Further reading

To find out more about the concepts mentioned above and ultimately how to set up custom events and contexts and send them to Snowplow pipeline, follow the links below.

- [Iglu common architecture](https://github.com/snowplow/iglu/wiki/Common-architecture)
- [Iglu clients](https://github.com/snowplow/iglu/wiki/Iglu-clients)
- [Iglu client configuration](https://github.com/snowplow/iglu/wiki/Iglu-client-configuration)
- [Iglu registries](https://github.com/snowplow/iglu/wiki/Iglu-repositories)
- [Iglu Central](https://github.com/snowplow/iglu/wiki/Iglu-Central-setup)
- [Self-describing JSON](https://github.com/snowplow/iglu/wiki/Self-describing-JSONs)
- [Self-describing JSON Schema](http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/)
- [SchemaVer](https://github.com/snowplow/iglu/wiki/SchemaVer)
- [Custom events](Custom-events)
- [Custom contexts](Custom-contexts)
- [Event dictionary](Event-dictionary)
