<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [Step 3: Setting up Enrich](Setting-up-enrich) > Configuring storage targets

Snowplow offers the option to configure certain storage targets.
This is done using configuration JSONs. 
When running EmrEtlRunner or StorageLoader, the `--targets` argument should be populated with the filepath of a directory containing your configuration JSONs.
Each storage target JSON file can have arbitrary name, but must conform it's JSON Schema.

Currently supported storage targets are:
Here's a list of currently supported targets, grouped by purpose:

* Enriched data

  * Redshift
  * PostgreSQL

* Failures

  * ElasticSearch

* Duplicate tracking

  * AWS DynamoDB

