[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 1: setup a Collector**](Setting-up-a-Collector) > [**Scala Stream Collector setup**](setting-up-the-Scala-Stream-Collector) > [[Configure the Scala Stream Collector]]

**This documentation refers to version 0.5.0 of the Scala Stream Collector**

**[Version 0.1.0][v0.1]**
**[Version 0.3.0 & 0.4.0][v0.3]**

The Scala Stream Collector has a number of configuration options available.

## Basic configuration

### Template

Download a template configuration file from GitHub: [config.hocon.sample][app-conf].

Now open the `config.hocon.sample` file in your editor of choice.

### AWS settings

Values that must be configured are:

+ `collector.aws.access-key`
+ `collector.aws.secret-key`

You can insert your actual credentials in these fields. Alternatively, if you set both fields to "env", your credentials will be taken from the environment variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY.

### Stream configuration

You will need to put in the names of your `good` and `bad` streams:

+ `collector.stream.good`
+ `collector.stream.bad`

If your stream's AWS region is not us-east-1, you should also change `collector.stream.region` correspondingly.

### HTTP settings

Also verify the settings of the HTTP service:

+ `collector.interface`
+ `collector.port`

### Buffer settings

You will also need to set appropriate limits for:

+ `collector.buffer.byte-limit`
+ `collector.buffer.record-limit`
+ `collector.buffer.time-limit`

## Additional configuration options

### 1. Setting the environment to develop or production

`collector.production` is a boolean that enables or disables production mode.
Production mode disables additional services helpful for configuring and
initializing the collector.

### 2. Sinks

The `collector.sink.enabled` setting determines which of the supported sinks to write raw events to:
+ `"kinesis`" for writing Thrift-serialized records and error rows to a Kinesis stream
+ `"stdout`" for writing Base64-encoded Thrift-serialized records and error rows to stdout and stderr respectively

The default setting is `"kinesis`".

If you switch to `"stdout`", we recommend changing the `akka: {}` section to prevent Akka/Spray debug information from polluting your event stream on stdout:

```
akka {
  loglevel = OFF
  loggers = []
}
```

### 3. Setting the P3P policy header

The P3P policy header is set in `collector.p3p`, and
if not set, the P3P policy header defaults to:

	policyref="/w3c/p3p.xml", CP="NOI DSP COR NID PSA OUR IND COM NAV STA"

### 4. Setting the domain name

Setting the domain name in `collector.cookie.domain` can be useful if you want to make the cookie accessible to other applications on your domain. In our example above, for example, we've setup the collector on `collector.snplow.com`. If we do not set a domain name, the cookie will default to this domain. However, if we set it to `.snplow.com`, that cookie will be accessible to other applications running on `*.snplow.com`.

Please, refer to [RFC 6265](https://tools.ietf.org/html/rfc6265#section-5.1.3) for the domain matching rules.

### 5. Setting the cookie duration

The cookie expiration duration is set in `collector.cookie.expiration`.
If no value is provided, cookies set default to expiring after one year (i.e. 365 days).
From version 0.4.0 forwards, if `collector.cookie.expiration` is set to 0, no cookie will be set.

Next: [[Run the Scala Stream collector]]

[v0.1]: https://github.com/snowplow/snowplow/wiki/Configure-the-Scala-Stream-Collector-v0.1
[v0.3]: https://github.com/snowplow/snowplow/wiki/Configure-the-Scala-Stream-Collector-v0.3
[app-conf]: https://github.com/snowplow/snowplow/blob/r67-bohemian-waxwing/2-collectors/scala-stream-collector/src/main/resources/config.hocon.sample
