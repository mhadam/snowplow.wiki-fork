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
- 4. [Example Gradle Dependencies](#example)
  - 4.1 [Classic](#classic)
  - 4.2 [RxJava](#rx-java)

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

To minimize the dex footprint of the Tracker we have kept dependencies to an absolute minimum.

To minimize jar bloat, we have tried to keep external dependencies to a minimum. For the full list of dependencies, please see our [core Gradle build file][gradle-build-core], [classic Gradle build file][gradle-build-classic] and our [rx Gradle build file][gradle-build-rx].

[Back to top](#top)

<a name="setup" />

## 3. Setup

<a name="hosting" />

### 3.1 Hosting

The Tracker is published to Snowplow's [hosted Maven repository][maven-snplow], which should make it easy to add it as a dependency into your own Android app.

The current version of the Snowplow Android Tracker is 0.5.4.

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

Then add into your project's `pom.xml` for the classic Tracker:

```xml
<dependency>
    <groupId>com.snowplowanalytics</groupId>
    <artifactId>snowplow-android-tracker-classic</artifactId>
    <version>0.5.4</version>
</dependency>
```

...and for the RxJava Tracker:

```xml
<dependency>
    <groupId>com.snowplowanalytics</groupId>
    <artifactId>snowplow-android-tracker-rx</artifactId>
    <version>0.5.4</version>
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

    // For Classic
    compile 'com.snowplowanalytics:snowplow-android-tracker-classic:0.5.4'

    // For RxJava
    compile 'com.snowplowanalytics:snowplow-android-tracker-rx:0.5.4'
}
```

This will install version `0.5.4` of the android tracker.  However if you would like to ensure that all bug fixes and patches for version `0.5.4` are installed, simply change `0.5.4` into `0.5.+`.

Please note that no breaking changes will occur in the '0.5.x' space.

```groovy
dependencies {
    ...
    // Snowplow Android Tracker

    // For Classic
    compile 'com.snowplowanalytics:snowplow-android-tracker-classic:0.5.+'

    // For RxJava
    compile 'com.snowplowanalytics:snowplow-android-tracker-rx:0.5.+'
}
```
<a name="aar" />

#### 3.3.1 Android Archive

You can also add the Android Tracker using the Android ARchive (aar) package in your gradle file in a similar way to the Gradle version while appending '@aar' to the end:

```groovy
dependencies {
    ...
    // Snowplow Android Tracker

    // For Classic
    compile 'com.snowplowanalytics:snowplow-android-tracker-classic:0.5.+@aar'

    // For RxJava
    compile 'com.snowplowanalytics:snowplow-android-tracker-rx:0.5.+@aar'
}
```

Please note that if using the `@aar` dependencies you will also need to include the core library, as this will not be brought in automatically.

```groovy
dependencies {
    ...
    // Snowplow Android Tracker

    // For Classic
    compile 'com.snowplowanalytics:snowplow-android-core:0.5.4@aar'
    compile 'com.snowplowanalytics:snowplow-android-tracker-classic:0.5.4@aar'

    // For RxJava
    compile 'com.snowplowanalytics:snowplow-android-core:0.5.4@aar'
    compile 'com.snowplowanalytics:snowplow-android-tracker-rx:0.5.4@aar'
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

<a name="example" />

## 4. Example Gradle Dependencies

<a name="classic" />

### 4.1 Classic

The dependencies for the Classic implementation of the Android Tracker goes as follows:

```groovy
repositories {
    maven {
        url "http://maven.snplow.com/releases"
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:support-v4:22.0.0'
    compile 'com.android.support:appcompat-v7:22.2.0'

    // Optional Google Analytics Library
    // - Required to get the IDFA Code
    compile 'com.google.android.gms:play-services-analytics:7.5.0'

    // Required Dependency for the Tracker
    compile 'com.squareup.okhttp:okhttp:2.1.0'

    // Tracker Import
    compile 'com.snowplowanalytics:snowplow-android-tracker-classic:0.5.4'
}
```

<a name="rx-java" />

### 4.2 RxJava

The dependencies for the RxJava implementation of the Android Tracker goes as follows:

```groovy
repositories {
    maven {
        url "http://maven.snplow.com/releases"
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:support-v4:22.0.0'
    compile 'com.android.support:appcompat-v7:22.2.0'

    // Optional Google Analytics Library
    // - Required to get the IDFA Code
    compile 'com.google.android.gms:play-services-analytics:7.5.0'

    // Required Dependency for the Tracker
    compile 'com.squareup.okhttp:okhttp:2.1.0'
    compile 'io.reactivex:rxjava:1.0.11'

    // Tracker Import
    compile 'com.snowplowanalytics:snowplow-android-tracker-rx:0.5.4'
}
```

Done? Now read the [Android Tracker API](Android-Tracker) to start tracking events.

[android]: http://www.android.com/

[gradle-build-core]: https://github.com/snowplow/snowplow-android-tracker/blob/master/snowplow-android-core/build.gradle
[gradle-build-classic]: https://github.com/snowplow/snowplow-android-tracker/blob/master/snowplow-android-tracker-classic/build.gradle
[gradle-build-rx]: https://github.com/snowplow/snowplow-android-tracker/blob/master/snowplow-android-tracker-rx/build.gradle
[maven-snplow]: http://maven.snplow.com
