[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » [**Step 3.1: setting up EmrEtlRunner**](Setting-up-EmrEtlRunner) » 5: Configuring shredding

1. [Overview](#overview)
2. [Pre-requisites](#pre-reqs)
3. [Configuring EmrEtlRunner](#configure)
4. [Next steps](#next-steps)

<a name="overview"/>

## 1. Overview

Snowplow has a [Shredding process](Shredding) for Redshift which contributes to the following three phases:

1. Extracting unstructured event JSONs and context JSONs from enriched event files into their own files
2. Removing endogenous duplicate records, which are sometimes introduced within the Snowplow pipeline (feature added to r76)
3. Loading those files into corresponding tables in Redshift

The first two phases are instrumented by EmrEtlRunner; in this page we will explain how to configure the shredding process to operate smoothly with EmrEtlRunner.

**Note: Even though the first phase is required only if you want to shred your own unstructured event JSONs and context JSONs, the second phase will be beneficial to data modeling and analysis. If none of it is required and you are only shredding Snowplow-authored JSONs like link clicks and ad impressions, then you can skip this page and go straight to [Loading shredded types](4-Loading-shredded-types).**

<a name="pre-reqs"/>

## 2. Pre-requisites

First off, we assume that all JSONs you are sending as unstructured events and contexts are self-describing JSONs. To find out more about self-describing JSONs:

* [Iglu documentation on self-describing JSONs](https://github.com/snowplow/iglu/wiki/Self-describing-JSONs)
* [JavaScript Tracker 2.0.0 release on self-describing JSONs](http://snowplowanalytics.com/blog/2014/07/03/snowplow-javascript-tracker-2.0.0-released/#schemas)
* [SchemaVer for semantic schema versioning](https://github.com/snowplow/iglu/wiki/SchemaVer)

Secondly, we assume that you have defined self-describing JSON Schemas for each of your JSONs. Resources:

* [Iglu documentation on self-describing JSON Schemas](https://github.com/snowplow/iglu/wiki/Self-describing-JSON-Schemas)
* [Understanding JSON Schema (PDF)](http://spacetelescope.github.io/understanding-json-schema/UnderstandingJSONSchema.pdf)

Thirdly, we assume that you have setup your own Iglu schema registry to host your schemas. Resources:

* [Setup a static Iglu repository](https://github.com/snowplow/iglu/wiki/Static-repo-setup)
* [Iglu FAQ: How do I host a schema privately?](https://github.com/snowplow/iglu/wiki/Developer-FAQ#how-do-i-host-a-schema-privately)

» Read more about the topics related to events and contexts:

- [Events and contexts](Events-and-contexts)
- [Iglu schema registry](Iglu-registry)

You are now ready to configure EmrEtlRunner for shredding.

<a name="configure"/>

## 3. Configuring EmrEtlRunner

The relevant section of the EmrEtlRunner's [`config.yml`](https://github.com/snowplow/snowplow/blob/master/3-enrich/emr-etl-runner/config/config.yml.sample) is:

```yaml
shredded:
  good: s3://my-out-bucket/shredded/good       # e.g. s3://my-out-bucket/shredded/good
  bad: s3://my-out-bucket/shredded/bad        # e.g. s3://my-out-bucket/shredded/bad
  errors: s3://my-out-bucket/shredded/errors     # Leave blank unless :continue_on_unexpected_error: set to true below
  archive: s3://my-out-bucket/shredded/archive  # Not required for Postgres currently
```

The configuration file is referenced with `--config` option to EmrEtlRunner.

Please make sure that these shredded buckets are set correctly.

Next, we let EmrEtlRunner know about your Iglu schema registry, so that schemas can be retrieved from there as well as from Iglu Central. Add your own registry to the repositories array in  [`iglu_resolver.json`](https://github.com/snowplow/snowplow/blob/master/3-enrich/config/iglu_resolver.json) file:

```yaml
{
  "schema": "iglu:com.snowplowanalytics.iglu/resolver-config/jsonschema/1-0-0",
  "data": {
    "cacheSize": 500,
    "repositories": [
      {
        "name": "Iglu Central",
        "priority": 0,
        "vendorPrefixes": [ "com.snowplowanalytics" ],
        "connection": {
          "http": {
            "uri": "http://iglucentral.com"
          }
        }
      }
      #custom section starts here -->
      ,
      {
       ...
      }
      #custom section ends here <--
    ]
  }
}
```

You must add an extra entr(-y/ies) in the `repositories:` array pointing to your own Iglu schema registry. If you are not submitting custom events and contexts and are not interested in shredding then there's no need in adding the custom section but the `iglu_resolver.json` file is still required and is referenced with `--resolver` option to EmrEtlRunner.

For more information on how to customize the `iglu_resolver.json` file, please review the [Iglu client configuration](https://github.com/snowplow/iglu/wiki/Iglu-client-configuration) wiki page.

<a name="next-steps"/>

## 4. Next steps

That's it for configuring EmrEtlRunner for shredding. Next, please refer to the [Loading shredded types](4-Loading-shredded-types) wiki page to understand how to configure StorageLoader.
