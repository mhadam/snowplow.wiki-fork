[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [Step 4: setting up alternative data stores](Setting-up-alternative-data-stores) » [1: Installing the StorageLoader](1-Installing-the-StorageLoader) » [2: Using the StorageLoader](2-Using-the-StorageLoader) » [3: Scheduling the StorageLoader](3-Scheduling-the-StorageLoader) » 4: Loading shredded types

1. [Overview](#overview)
2. [Loading Snowplow-authored JSONs](#snowplow-jsons)
3. [Defining and installing a new table](#table)
4. [Creating and uploading a JSON Paths file](#jsonpaths)
5. [Configuring StorageLoader](#configure)
6. [Next steps](#next-steps)

<a name="overview"/>

## 1. Overview

Snowplow has a [Shredding process](Shredding) for Redshift which consists of three phases:

1. Extracting unstructured event JSONs and context JSONs from enriched event files into their own files
2. Removing endogenous duplicate records, which are sometimes introduced within the Snowplow pipeline (feature added to r76)
3. Loading those files into corresponding tables in Redshift

The third phase is instrumented by StorageLoader and is documented on this wiki page; to configure the first phase, visit the [Configuring shredding](5-Configuring-shredding) wiki page (second phase requires no configuration).

<a name="snowplow-jsons"/>

## 2. Loading Snowplow-authored JSONs

Loading Snowplow-authored JSONs is straightforward: Snowplow provides pre-made Redshift table definitions for all Snowplow-authored JSONs. You can find these here:

    https://github.com/snowplow/snowplow/tree/master/4-storage/redshift-storage/sql

For example, if you have link click tracking enabled in the JavaScript Tracker, then install `com.snowplowanalytics.snowplow/link_click_1.sql` into your Snowplow database.

Each table needs to be loaded using a JSON Paths file. Snowplow hosts JSON Paths files for all Snowplow-authored JSONs. StorageLoader will automatically locate these JSON Paths files and use them to load shredded types into Redshift.

<a name="overview"/>

## 3. Defining and installing a new table

### 3.1 Overview

StorageLoader loads each shredded type into its own table in Redshift. You need to create a Redshift table for each new shredded type you have defined.

### 3.2 Naming the table

The table name must be a SQL-friendly compound of the schema's vendor, name and model version, converted from CamelCase to snake_case and with any periods or hyphens replaced with underscores. For example, with the Iglu schema key:

    iglu:com.acme.website/anonymous-customer/jsonschema/1-0-2

The table name would be:

    com_acme_website_anonymous_customer_1

With the Iglu schema key:

    iglu:de.company/AddToBasket/jsonschema/2-1-0

The table name would be:

    de_company_add_to_basket_2

Note that only the model version is included - do not include the remaining portions of the version (SchemaVer revision or addition).

### 3.2 Creating the table definition

Each table definition starts with a set of standard "boilerplate" fields. These fields help to document the type hierarchy which has been shredded and are very useful for later analysis. These fields are as follows:

```sql
CREATE TABLE atomic.com_snowplowanalytics_snowplow_link_click_1 (
    -- Schema of this type
    schema_vendor   varchar(128)  encode runlength not null,
    schema_name     varchar(128)  encode runlength not null,
    schema_format   varchar(128)  encode runlength not null,
    schema_version  varchar(128)  encode runlength not null,
	-- Parentage of this type
	root_id         char(36)      encode raw not null,
	root_tstamp     timestamp     encode raw not null,
	ref_root        varchar(255)  encode runlength not null,
	ref_tree        varchar(1500) encode runlength not null,
	ref_parent      varchar(255)  encode runlength not null,
	...
```

Now you can add the fields required for your JSON:

```sql
	...
	-- Properties of this type
	element_id      varchar(255)  encode text32k,
	element_classes varchar(2048) encode raw,
	element_target  varchar(255)  encode text255,
	target_url      varchar(4096) encode text32k not null
	...
```

Note that, in the example above, `element_classes` was originally a JSON array in the source JSON. Because our Shredding process does not yet support nested shredding, we simply set this field to a large varchar; an analyst can use Redshift's in-built JSON support to explore this field's contents.

And finally, all tables should have a standard `DISTKEY` and `SORTKEY`:

```sql
    ...
)
DISTSTYLE KEY
-- Optimized join to atomic.events
DISTKEY (root_id)
SORTKEY (root_tstamp);
```

These keys are designed to make JOINs from these tables back to `atomic.events` as performant as possible.

### 3.3 Installing the table

Install the table into your Redshift database. The table must be stored in the same schema as your `events` table.

<a name="jsonpaths"/>

## 4. Creating and uploading a JSON Paths file

### 4.1 Overview

You need to create a JSON Paths file which StorageLoader will use to load your shredded type into Redshift.

The format is simple - a JSON Paths file consists of a JSON array, where each element corresponds to a column in the target table. For full details, see the [Copy from JSON](http://docs.aws.amazon.com/redshift/latest/dg/copy-usage_notes-copy-from-json.html) documentation from Amazon.

### 4.2 Creating a JSON Paths file

To correspond to the table definition, each JSON Path file must start with the following elements:

```json
{
  "jsonpaths": [

    "$.schema.vendor",
    "$.schema.name",
    "$.schema.format",
    "$.schema.version",

    "$.hierarchy.rootId",
    "$.hierarchy.rootTstamp",
    "$.hierarchy.refRoot",
    "$.hierarchy.refTree",
    "$.hierarchy.refParent",
    ...
```

Then finish your array of JSON Paths with an element for each custom field in your table. For example, here are the remaining fields for the Snowplow link click event:

```json
	...
    "$.data.elementId",
    "$.data.elementClasses",
    "$.data.elementTarget",
    "$.data.targetUrl"
    ]
}
```

A few things to note:

* Relational Database Shredder will nest all of your JSON's properties into a `data` property, hence the namespacing seen above
* Currently the Shredding process does not support nested tables. A nested property such as an array (like `elementClasses` above) should be loaded into a single field

### 4.3 Naming the JSON Paths file

The JSON Paths file should be named the same as the table created in 3, **minus** the shredded type's vendor. For example, if your table is called:

    com_acme_website_anonymous_customer_1

Then your JSON Paths file should be called:

    anonymous_customer_1.json

### 4.4 Installing the JSON Paths file

Upload the JSON Paths file to a private S3 bucket which is accessible using the AWS credentials provided in your StorageLoader's `config.yml` file.

Store the JSON Paths file in a sub-folder named after the vendor, for example:

    s3://acme-jsonpaths-files/com.acme.website/anonymous_customer_1.json

<a name="configure"/>

## 5. Configuring StorageLoader

Now you need to update StorageLoader's `config.yml` to load the shredded types. First, make sure that your `jsonpath_assets:` points to the private S3 bucket you stored the JSON Paths file in section 4.

```yaml
buckets:
  jsonpath_assets: s3://acme-jsonpaths-file
```

Next, make sure that you have populated the `shredded:` section correctly:

```yaml
shredded:
  good: s3://my-data-bucket/shredded/good       # e.g. s3://my-out-bucket/shredded/good
  bad: s3://my-data-bucket/shredded/bad        # e.g. s3://my-out-bucket/shredded/bad
  errors: s3://my-data-bucket/shredded/errors     # Leave blank unless :continue_on_unexpected_error: set to true below
  archive: s3://my-data-bucket/shredded/archive    # Where to archive shredded events to, e.g. s3://my-archive-bucket/shredded
```

If you are using separate configuration files make sure this cross-checks with the corresponding paths in your EmrEtlRunner's `config.yml`.

<a name="next-steps"/>

## 6. Next steps

That's it for configuring StorageLoader for shredding. You should be ready to load shredded types into Redshift.
