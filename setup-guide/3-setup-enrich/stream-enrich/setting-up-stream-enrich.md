<a name="top" />

[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » Step 3.2: setting up Stream Enrich

[[https://d3i6fms1cm1j0i.cloudfront.net/github-wiki/images/3-enrich.png]] 

## Overview of Stream Enrich

[Stream Enrich][stream-enrich] is an application which:

1. **Reads** raw Snowplow events off a stream populated by the Scala Stream Collector
2. **Validates** each raw event
2. **Enriches** each event (e.g. infers the location of the user from his/her IP address)
3. **Writes** the enriched Snowplow event to another stream

This guide covers how to setup Stream Enrich, specifically:

1. [Installation](Install-Stream-Enrich) - you need to install Stream Enrich on your own server
2. [Configuration](Configure-Stream-Enrich) - how to use Stream Enrich at the command line, to instruct it to process data from your collector
3. [Running](Run-Stream-Enrich) - how to run Stream Enrich

[stream-enrich]: https://github.com/snowplow/snowplow/tree/master/3-enrich/stream-enrich
