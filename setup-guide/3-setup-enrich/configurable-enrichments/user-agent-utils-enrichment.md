<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [Step 3: Setting up Enrich](Setting-up-enrich) > [Configurable enrichments](Configurable-enrichments) > user-agent-utils enrichment

### Compatibility

JSON Schema   [iglu:com.snowplowanalytics.snowplow/user_agent_utils_config/jsonschema/1-0-0][schema]
Compatibility r63+
Data provider [user-agent-utils][user-agent-utils]

### Overview

This enrichment uses the [user-agent-utils][user-agent-utils] library to parse the useragent into the following fields:

`br_name`
`br_family`
`br_version`
`br_type`
`br_renderengine`
`os_name`
`os_family`
`os_manufacturer`
`dvce_type`
`dvce_ismobile`

### Example

This enrichment has no special configuration: it is either off or on. Enable it with the following JSON:

```json
{
    "schema": "iglu:com.snowplowanalytics.snowplow/user_agent_utils_config/jsonschema/1-0-0",
    "data": {
        "vendor": "com.snowplowanalytics.snowplow",
        "name": "user_agent_utils_config",
        "enabled": true,
        "parameters": {}
  }
}
```

***Note***: As an alternative solution, you could enable [`ua-parser` enrichment](ua-parser-enrichment) either in place of this enrichment or as an accopmanying enhancement. There's no conflict here as the output data of these enrichments will end up in different tables.

### Data sources

The input value for the enrichment comes from `ua` parameter which is mapped to `useragent` field in `atomic.events` table.

### Algorithm

This enrichment uses 3rd party [`user-agent-utils`][user-agent-utils] library to parse the useragent string.

### Data generated

Below is the summary of the fields in `atomic.events` table driven by the result of this enrichment (no dedicated table).

Field | Purpose
:---|:---
`br_name` | Browser name
`br_family` | Browser family
`br_version` | Browser version
`br_type` | Browser type
`br_renderengine` | Rendering engine of the browserw
`os_name` | Operationg sytem name
`os_family` | Operationg sytem family
`os_manufacturer` | Manufacturers of operating system
`dvce_type` | Device type
`dvce_ismobile` | Indicates whether divice is moblile

[schema]: http://iglucentral.com/schemas/com.snowplowanalytics.snowplow/user_agent_utils_config/jsonschema/1-0-0
[user-agent-utils]: https://github.com/HaraldWalker/user-agent-utils
