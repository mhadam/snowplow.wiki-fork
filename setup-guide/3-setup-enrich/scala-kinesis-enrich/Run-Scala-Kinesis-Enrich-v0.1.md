<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [Step 3: Setting up Enrich](Setting-up-enrich) > [**Step 3.2: setting up Scala Kinesis Enrich**](Setting-up-Scala-Kinesis-Enrich) > [3: Running](Running-Scala-Kinesis-Enrich)

**This page refers to version 0.1.0 of Scala Kinesis Enrich. Documentation for other versions is available:**

**[Version 0.3][v0.3]**

## Running

Scala Kinesis Enrich is an executable jarfile which should be runnable from any Unix-like shell environment. Simply provide the configuration file as a parameter:

    $ ./scala-kinesis-enrich-0.1.0 --config my.conf

This will start the Scala Kinesis Enrich app to read raw events from Kinesis and write enriched events back to Kinesis.

## All done?

You have setup Scala Kinesis Enrich! You are now ready to [setup alternative data stores](Setting-up-alternative-data-stores).

Return to the [setup guide](Setting-up-Snowplow).

[v0.3]: https://github.com/snowplow/snowplow/wiki/Run-Scala-Kinesis-Enrich-v0.3
