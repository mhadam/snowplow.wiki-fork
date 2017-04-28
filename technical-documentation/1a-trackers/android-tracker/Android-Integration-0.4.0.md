<a name="top" />

This page refers to integration samples for [Android Tracker v0.4.0][android-tracker].  This assumes you have already successfully gone through the [Setup Guide][setup-guide] for setting up all the dependencies for the tracker.

The following example classes are using the bare minimum of settings for building a Tracker.  It is encouraged to flesh out the options for the Tracker and Emitter builders.

## Contents

- 1. [Classic Tracker](#classic)
- 2. [RxJava Tracker](#rx-java)
- 3. [Tracking Events](#events)

<a name="classic" />

## 1. Classic Tracker

You will need to have imported the following library into your project:

```groovy
dependencies {
    ...
    // Snowplow Android Tracker Classic
    compile 'com.snowplowanalytics:snowplow-android-tracker-classic:0.4.0'
}
```

Example class to return an Android Classic Tracker:

```java
import com.snowplowanalytics.snowplow.tracker.*;
import android.content.Context;

public class TrackerBuilderClassic {

    public static Tracker getClassicTracker(Context context) {
        Emitter emitter = getClassicEmitter(context);
        Subject subject = getSubject(context);  // Optional

        return new Tracker
            .TrackerBuilder(emitter, "your-namespace", "your-appid",
                com.snowplowanalytics.snowplow.tracker.classic.Tracker.class)
            .subject(subject) // Optional
            .build();
    }

    private static Emitter getClassicEmitter(Context context) {
        return new Emitter
            .EmitterBuilder("notarealuri.fake.io", context,
                com.snowplowanalytics.snowplow.tracker.classic.Emitter.class)
            .build();
    }

    private static Subject getSubject(Context context) {
        return new Subject
            .SubjectBuilder()
            .context(context)
            .build();
    }
}
```

[Back to top](#top)

<a name="rx-java" />

## 2. RxJava tracker

You will need to have imported the following library into your project:

```groovy
dependencies {
    ...
    // Snowplow Android Tracker Rx
    compile 'com.snowplowanalytics:snowplow-android-tracker-rx:0.4.0'
}
```

Example class to return an Android RxJava Tracker:

```java
import com.snowplowanalytics.snowplow.tracker.*;
import android.content.Context;

public class TrackerBuilderRx {

    public static Tracker getRxTracker(Context context) {
        Emitter emitter = getRxEmitter(context);
        Subject subject = getSubject(context); // Optional

        return new Tracker
            .TrackerBuilder(emitter, "your-namespace", "your-appid",
                com.snowplowanalytics.snowplow.tracker.rx.Tracker.class)
            .subject(subject) // Optional
            .build();
    }

    private static Emitter getRxEmitter(Context context) {
        return new Emitter
            .EmitterBuilder("notarealuri.fake.io", context,
                com.snowplowanalytics.snowplow.tracker.rx.Emitter.class)
            .build();
    }

    private static Subject getSubject(Context context) {
        return new Subject
            .SubjectBuilder()
            .context(context)
            .build();
    }
}
```

[Back to top](#top)

<a name="events" />

## 3. Tracking Events

Once you have successfully built your Tracker object you can track events with calls like the following:

```java
Tracker tracker = getClassicTracker(context);
tracker.trackScreenView("someScreenName", "someScreenId");
```

For an outline of all available tracking combinations have a look [here][demo-app-track-events].

[Back to top](#top)

[android-tracker]: https://github.com/snowplow/snowplow/wiki/Android-Tracker-0.4.0
[setup-guide]: https://github.com/snowplow/snowplow/wiki/Android-Tracker-Setup-0.4.0
[demo-app-track-events]: https://raw.githubusercontent.com/snowplow/snowplow-android-tracker/0.4.0/snowplow-demo-app/src/main/java/com/snowplowanalytics/snowplowtrackerdemo/utils/TrackerEvents.java
