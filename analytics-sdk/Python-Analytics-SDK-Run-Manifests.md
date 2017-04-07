<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Snowplow Analytics SDK**](Snowplow-Analytics-SDK) » Python Analytics SDK Run Manifests

**This feature is added in 0.2.0 release, which is not published yet**

## Overview

The [Snowplow Analytics SDK for Python](https://github.com/snowplow/snowplow-python-analytics-sdk) provides you an API to work with run manifests.
Run manifests is simple way to mark chunk (particular run) of enriched data as being processed, by for example Apache Spark data-modeling job.

## Usage

Run manifests functionality resides in new `snowplow_analytics_sdk.run_manifests` module.

Main class is `RunManifests`, that proides access to DynamoDB table via `contains` and `add`, as well as `create` method to initialize table with appropriate settings.
Other commonly-used function is `list_runids` that is gives S3 client and path to folder such as `enriched.archive` or `shredded.archive` from `config.yml` lists all 
folders that match Snowplow run id format (`run-YYYY-mm-DD-hh-MM-SS`).
Using `list_runids` and `RunManifests` you can list job runs and safely process them one by one without risk of reprocessing.

## Example

Here's a short usage example:

```python
from boto3 import client
from snowplow_analytics_sdk.run_manifests import *

s3 = client('s3')
dynamodb = client('dynamodb')

dynamodb_run_manifests_table = 'snowplow-run-manifests'
enriched_events_archive = 's3://acme-snowplow-data/storage/enriched-archive/'
run_manifests = RunManifests(dynamodb, dynamodb_run_manifests_table)

run_manifests.create()   # This should be called only once

for run_id in list_runids(s3, enriched_events_archive):
    if not run_manifests.contains(run_id):
        process(run_id)
        run_manifests.add(run_id)
    else:
        pass
```

In above example, we create two AWS service clients for S3 (to list job runs) and for DynamoDB (to access manifests).
These cliens are provided via [boto3][boto3] Python AWS SDK and can be initialized with static credentials or with system-provided credentials.

Then we list all run ids in particular S3 path and process (by user-provided `process` function) only those that were not processed already.
Note that `run_id` is simple string with S3 key of particular job run.

`RunManifests` class is a simple API wrapper to DynamoDB, using which you can:

* `create` DynamoDB table for manifests, 
* `add` run to table 
* check if table `contains` run id

[Back to top](#top)  
[Back to Python Analytics SDK contents][contents]

[contents]: Python-Analytics-SDK
