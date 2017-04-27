[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 1: setup a Collector**](Setting-up-a-Collector) > [**Scala Stream Collector setup**](setting-up-the-Scala-Stream-Collector) > [[Configure the Scala Stream Collector]]

*This documentation refers to version 0.1.0 of the Scala Stream Collector.*

*[Version 0.2.0][v0.2]*

The Scala Stream Collector has a number of configuration options available.

## Basic configuration

### Template

Download a template configuration file from GitHub: [config.hocon.sample][app-conf].

Now open the `config.hocon.sample` file in your editor of choice.

### AWS settings

Values that must be configured are:

+ `collector.aws.access-key`
+ `collector.aws.secret-key`

### Stream configuration

We also recommend changing the default stream options:

+ `collector.stream.name`
+ `collector.stream.size`

### HTTP settings

Also verify the settings of the HTTP service:

+ `collector.interface`
+ `collector.port`

## Additional configuration options

### 1. Setting the environment to develop or production

`collector.production` is a boolean that enables or disables production mode.
Production mode disables additional services helpful for configuring and
initializing the collector.

### 2. Sinks

The `collector.sink.enabled` setting determines which of the supported sinks to write raw events to:
+ `"kinesis`" for writing Thrift-serialized records to a Kinesis stream
+ `"stdout`" for writing Base64-encoded Thrift-serialized records to stdout

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

Next: [[Run the Scala Stream collector]]

[v0.2]: [https://github.com/snowplow/snowplow/wiki/Configure-the-Scala-Stream-Collector]
[app-conf]: https://github.com/snowplow/snowplow/blob/master/2-collectors/scala-stream-collector/src/main/resources/config.hocon.sample
