<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**Android tracker**](Java-tracker-setup)

## Contents

- 1 [Overview](#overview)  
- 2 [Integration options](#integration-options)
  - 2.1 [Tracker compatibility](#compatibility)  
  - 2.2 [Dependencies](#dependencies)
- 3 [Setup](#setup)
  - 3.1 [Hosting](#hosting)
  - 3.2 [Gradle](#gradle)
  - 3.3 [Permissions](#permissions)
  - 3.4 [Proguard](#proguard)
- 4 [Example Gradle Dependencies](#example)

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

The Snowplow Android Tracker has been built and tested using the Android SDK version 24, but uses a minimum SDK version of 11, so should work within any Android application built using SDK version 11 upwards.

[Back to top](#top)

<a name="dependencies" />

### 2.2 Dependencies

To minimize the dex footprint of the Tracker we have kept dependencies to an absolute minimum.  

To minimize jar bloat, we have tried to keep external dependencies to a minimum.  The current dependencies:

```
compile 'com.squareup.okhttp3:okhttp:3.4.1'
```

[Back to top](#top)

<a name="setup" />

## 3. Setup

<a name="hosting" />

### 3.1 Hosting

The Tracker is published to jcenter, which should make it easy to add it as a dependency into your own Android app.

The current version of the Snowplow Android Tracker is 0.6.2.

<a name="gradle" />

### 3.2 Gradle

If you are using Gradle in your own Android application, you will need to ensure `jcenter()` is in your `build.gradle` file:

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
    // Snowplow Android Tracker
    compile 'com.snowplowanalytics:snowplow-android-tracker:0.6.2@aar'
}
```

This will install version `0.6.2` of the android tracker.  However if you would like to ensure that all bug fixes and patches for version `0.6.2` are installed, simply change `0.6.2` into `0.6.+`.  

Please note that no breaking changes will occur in the '0.6.x' space.

```groovy
dependencies {
    ...
    // Snowplow Android Tracker
    compile 'com.snowplowanalytics:snowplow-android-tracker:0.6.+@aar'
}
```

<a name="permissions" />

### 3.3 Permissions

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

<a name="proguard" />

### 3.4 Proguard

If you enable Proguard in your application (`minifyEnabled true`) add the following rules to the Proguard configuration file

```
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient {*;} 
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient$Info {*;}
```

<a name="example" />

## 4. Example Gradle Dependencies

The dependencies for the implementation of the Android Tracker goes as follows:

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
    compile 'com.squareup.okhttp3:okhttp:3.4.1'

    // Tracker Import
    compile 'com.snowplowanalytics:snowplow-android-tracker:0.6.2@aar'
}
```

Done? Now read the [Android Tracker API](Android-Tracker) to start tracking events.

[android]: http://www.android.com/
[maven-snplow]: http://maven.snplow.com
