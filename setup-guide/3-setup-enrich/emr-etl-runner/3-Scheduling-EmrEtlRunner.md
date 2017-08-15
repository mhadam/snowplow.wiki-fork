<a name="top" />

[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » [Step 3.1: setting up EmrEtlRunner](Setting-up-EmrEtlRunner) [Step 3.1.1: Installing EmrEtlRunner](1-Installing-EmrEtlRunner) » [Step 3.1.2: Using EmrEtlRunner](2-Using-EmrEtlRunner) » Step 3.1.3: Scheduling EmrEtlRunner

1. [Overview](#scheduling-overview)
2. [cron](#cron)
3. [Jenkins](#jenkins)
4. [Windows Task Scheduler](#windows)
5. [Next steps](#next-steps)

<a name="scheduling"/>

## Scheduling

<a name="scheduling-overview"/>

## 1. Overview

Once you have the ETL process working smoothly, you can schedule a daily
(or more frequent) task to automate the daily ETL process.

We run our daily ETL jobs at 3 AM UTC so that we are sure that we have
processed all of the events from the day before (CloudFront logs can
take some time to arrive).

To consider your different scheduling options in turn:

<a name="cron"/>

## 2. cron

[[/images/warning.png]] | Running EmrEtlRunner as *Ruby* (rather than *JRuby* apps) is no longer actively supported. The latest version of the EmrEtlRunner is available from our Bintray [here](http://dl.bintray.com/snowplow/snowplow-generic/snowplow_emr_r91_stonehenge.zip).
---|:---

The recommended way of scheduling the ETL process is as a daily cronjob.

    0 4   * * *   root    cronic /path/to/eer/snowplow-emr-etl-runner run -c config.yml

This will run the ETL job daily at 4 AM, emailing any failures to you via cronic.

<a name="jenkins"/>

## 3. Jenkins

Some developers use the [Jenkins][jenkins] continuous integration server (or
[Hudson][hudson], which is very similar) to schedule their Hadoop and Hive jobs.

Describing how to do this is out of scope for this guide, but the blog post
[Lowtech Monitoring with Jenkins][jenkins-tutorial] is a great tutorial on using
Jenkins for non-CI-related tasks, and could be easily adapted to schedule
EmrEtlRunner.

<a name="windows"/>

## 4. Windows Task Scheduler

For Windows servers, in theory it should be possible to use a Windows PowerShell
script plus [Windows Task Scheduler][windows-task-scheduler] instead of bash and cron. However, this has not been tested or documented.

If you get this working, please let us know!

<a name="next-steps" />

## 5. Next steps

Now you have installed and scheduled [EmrEtlRunner][emr-etl-runner], you have all your data ready for analysis in S3. Learn how to [setup the StorageLoader][storage-loader] to regularly load your data into a database e.g. Infobright or Redshift for e.g. OLAP analysis, or to [analyse it on S3 via Emr][emr-analysis].


[emr-etl-runner]: https://github.com/snowplow/snowplow/tree/master/3-enrich/emr-etl-runner
[hive-etl]: https://github.com/snowplow/snowplow/tree/master/3-enrich/hive-etl
[trackers]: https://github.com/snowplow/snowplow/tree/master/1-trackers
[collectors]: https://github.com/snowplow/snowplow/tree/master/2-collectors
[getting-started]: http://snowplowanalytics.com/product/get-started.html

[git-install]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[ruby-install]: http://www.ruby-lang.org/en/downloads/
[nokogiri-install]: http://nokogiri.org/tutorials/installing_nokogiri.html
[rubygems-install]: http://docs.rubygems.org/read/chapter/3

[config-yml]: https://github.com/snowplow/snowplow/tree/master/3-enrich/emr-etl-runner/config/config.yml
[bash-script]: https://github.com/snowplow/snowplow/blob/r76-changeable-hawk-eagle/3-enrich/emr-etl-runner/bin/snowplow-emr-etl-runner.sh

[cronic]: http://habilis.net/cronic/
[jenkins]: http://jenkins-ci.org/
[hudson]: http://hudson-ci.org/
[jenkins-tutorial]: http://blog.lusis.org/blog/2012/01/23/lowtech-monitoring-with-jenkins/
[windows-task-scheduler]: http://en.wikipedia.org/wiki/Windows_Task_Scheduler#Task_Scheduler_2.0

[storage-loader]: https://github.com/snowplow/snowplow/wiki/Setting-up-Snowplow#wiki-step4
[emr-analysis]: https://github.com/snowplow/snowplow/wiki/Setting-up-Snowplow#wiki-step5
