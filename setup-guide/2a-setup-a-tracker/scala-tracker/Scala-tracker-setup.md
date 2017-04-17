<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**Scala tracker**](Scala-tracker-setup)

## Contents

- 1 [Overview](#overview)  
- 2 [Setup](#setup)
  - 2.1 [Hosting](#hosting)
  - 2.2 [SBT](#sbt)
  - 2.3 [Gradle](#gradle)
  - 2.4 [Maven](#maven)
- 3 [Older versions](#older)


<a name="overview" />

## 1. Overview

The [Snowplow Scala Tracker](https://github.com/snowplow/snowplow-scala-tracker) lets you add analytics to your Scala apps and servers.

Setting up the tracker should be relatively straightforward if you are familiar with [sbt][sbt].

<a name="setup" />

## 2. Setup

<a name="hosting" />

### 2.1 Hosting

The Tracker is published to Maven Central and JCenter, which should make it easy to add it as a dependency into your own Scala app.

<a name="sbt" />

### 2.2 SBT

Add the Scala Tracker to your build.sbt like this:

```scala
resolvers += "JCenter" at "https://jcenter.bintray.com/" // you can omit if you're planning to use Maven Central

libraryDependencies += "com.snowplowanalytics" %% "snowplow-scala-tracker" % "0.3.0"
```

<a name="gradle" />

### 2.3 Gradle

If you are using Gradle in your own Scala application, then add our Maven repository in your `build.gradle` file:

```groovy
repositories {
    ...
    jcenter()
}
```

Then add into the same file:

```groovy
dependencies {
    ...
    // Snowplow Scala Tracker
    compile 'com.snowplowanalytics:snowplow-scala-tracker_2.10:0.3.0'
}
```

Notice a `_2.10` postfix in artifactId. This is used for Scala libraries and denote Scala version which artifact (in our case `snowplow-scala-tracker`) is compiled against. It also means that this library will bring a `org.scala-lang:scala-library_2.10.x` as transitive dependency and if you're using any other Scala dependency you should keep these postfixes in accordance (`snowplow-scala-tracker` is also compiled against Scala 2.11).

<a name="maven" />

### 2.4 Maven

If you are using Maven for building your Scala application, then add the following code into your `HOME/.m2/settings.xml` to be able to use this repository:

```xml
<settings>
  <profiles>
    <profile>
      <!-- ... -->
      <repositories>
        <repository>
          <id>central</id>
          <name>bintray</name>
          <url>https://jcenter.bintray.com/</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <id>central</id>
          <name>bintray-plugins</name>
          <url>http://jcenter.bintray.com</url>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
  <activeProfiles>
    <activeProfile>bintray</activeProfile>
  </activeProfiles>
</settings>
```

Then add into your project's `pom.xml`:

```xml
<dependency>
    <groupId>com.snowplowanalytics</groupId>
    <artifactId>snowplow-scala-tracker_2.10</artifactId>
    <version>0.2.0</version>
</dependency>
```

Notice a `_2.10` postfix in artifactId. This is used for Scala libraries and denote Scala version which artifact (in our case `snowplow-scala-tracker`) is compiled against. It also means that this library will bring a `org.scala-lang:scala-library_2.10.x` as transitive dependency and if you're using any other Scala dependency you should keep these postfixes in accordance (`snowplow-scala-tracker` is also compiled against Scala 2.11).

<a name="older" />

### 3 Older versions

Scala Tracker is published on JCenter and Maven Central since version 0.3.0. 
Previous versions can be found on Snowplow's [hosted Maven repository][maven-snplow].

Done? Now read the [Scala Tracker API](Scala-Tracker) to start tracking events.

[Back to top](#top)

[sbt]: http://www.scala-sbt.org/
[maven-snplow]: http://maven.snplow.com 