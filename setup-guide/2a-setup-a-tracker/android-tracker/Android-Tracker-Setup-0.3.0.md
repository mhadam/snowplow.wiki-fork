<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**Android tracker**](Java-tracker-setup)

## Contents

- 1. [Overview](#overview)
- 2. [Integration options](#integration-options)
  - 2.1 [Tracker compatibility](#compatibility)
  - 2.2 [Dependencies](#dependencies)
- 3. [Setup](#setup)
  - 3.1 [Hosting](#hosting)
  - 3.2 [Maven](#maven)
  - 3.3 [Gradle](#gradle)
    - 3.3.1 [Android Archive](#aar)
  - 3.4 [Permissions](#permissions)

<a name="overview" />

## 1. Overview

The [Snowplow Android Tracker](https://github.com/snowplow/snowplow-android-tracker) lets you add analytics to your [Android][android]-based mobile apps and games.

The Tracker should be relatively straightforward to setup if you are familiar with Java/Android development.

Ready? Let's get started.

[Back to top](#top)

<a name="integration-options" />

## 2. Integration options

<a name="compatibility" />

### 2.1 Tracker compatibility

The Snowplow Android Tracker has been built and tested using the Android SDK version 21, but uses a minimum SDK version of 11, so should work within any Android application built using SDK version 11 upwards.

[Back to top](#top)

<a name="dependencies" />

### 2.2 Dependencies

To minimize jar bloat, we have tried to keep external dependencies to a minimum. For the full list of dependencies, please see our [Gradle build file][gradle-build].

[Back to top](#top)

<a name="setup" />

## 3. Setup

<a name="hosting" />

### 3.1 Hosting

The Tracker is published to Snowplow's [hosted Maven repository][maven-snplow], which should make it easy to add it as a dependency into your own Android app.

The current version of the Snowplow Android Tracker is 0.3.0.

<a name="maven" />

### 3.2 Maven

If you are using Maven for building your Android application, then add the following code into your `HOME/.m2/settings.xml` to be able to use this repository:

```xml
<settings>
  <profiles>
    <profile>
      <!-- ... -->
      <repositories>
        <repository>
          <id>com.snowplowanalytics</id>
          <name>SnowPlow Analytics</name>
          <url>http://maven.snplow.com/releases</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
        </repository>
      </repositories>
    </profile>
  </profiles>
</settings>
```

Then add into your project's `pom.xml`:

```xml
<dependency>
    <groupId>com.snowplowanalytics</groupId>
    <artifactId>snowplow-android-tracker</artifactId>
    <version>0.3.0</version>
</dependency>
```

<a name="gradle" />

### 3.3 Gradle

If you are using Gradle in your own Android application, then add our Maven repository in your `build.gradle` file:

```groovy
repositories {
    ...
    maven {
        url "http://maven.snplow.com/releases"
    }
}
```

Then add into the same file:

```groovy
dependencies {
    ...
    // Snowplow Android Tracker
    compile 'com.snowplowanalytics:snowplow-android-tracker:0.3.0'
}
```

This will install version `0.3.0` of the android tracker.  However if you would like to ensure that all bug fixes and patches for version `0.3.0` are installed, simply change `0.3.0` into `0.3.+`.

Please note that no breaking changes will occur in the '0.3.x' space.

```groovy
dependencies {
    ...
    // Snowplow Android Tracker
    compile 'com.snowplowanalytics:snowplow-android-tracker:0.3.+'
}
```
<a name="aar" />

#### 3.3.1 Android Archive

You can also add the Android Tracker using the Android ARchive (aar) package in your gradle file in a similar way to the Gradle version while appending '@aar' to the end:
```groovy
dependencies {
    ...
    // Snowplow Android Tracker
    compile 'com.snowplowanalytics:snowplow-android-tracker:0.3.+@aar'
}
```

<a name="permissions" />

### 3.4 Permissions

To send the events, you need to update your `AndroidManifest.xml` with the internet access permission:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

To have the emitter check for online status before sending you will need to add the following:

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
```

If you want to send location information with each event you will need to add the following permissions to your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

Done? Now read the [Android Tracker API](Android-Tracker) to start tracking events.

[android]: http://www.android.com/

[gradle-build]: https://github.com/snowplow/snowplow-android-tracker/blob/master/snowplow-android-tracker/build.gradle
[maven-snplow]: http://maven.snplow.com
