<a name="top" />

This page refers to integration samples for [Android Tracker v0.6.0][android-tracker].  This assumes you have already successfully gone through the [Setup Guide][setup-guide] for setting up all the dependencies for the tracker.

The following example classes are using the bare minimum of settings for building a Tracker.  It is encouraged to flesh out the options for the Tracker and Emitter builders.

For previous versions:

* [v0.5.0][0.5.0]
* [v0.4.0][0.4.0]

## Contents

- 1. [Init Tracker](#tracker)
- 2. [Tracking Events](#events)
- 3. [Application Focus](#app-focus)

<a name="tracker" />

## 1. Init Tracker

You will need to have imported the following library into your project:

```groovy
dependencies {
    ...
    compile 'com.snowplowanalytics:snowplow-android-tracker:0.6.0@aar'
}
```

Example class to return an Android Tracker:

```java
import com.snowplowanalytics.snowplow.tracker.*;
import android.content.Context;

public class SnowplowTrackerBuilder {

    public static Tracker getTracker(Activity activity, Context context) {
        Emitter emitter = getEmitter(context);
        Subject subject = getSubject(context); // Optional

        return Tracker.init(new Tracker.TrackerBuilder(emitter, "your-namespace", "your-appid", context)
            .subject(subject) // Optional
            .build()
        ).setLifecycleHandler(activity);
    }

    private static Emitter getEmitter(Context context) {
        return new Emitter.EmitterBuilder("notarealuri.fake.io", context)
            .build();
    }

    private static Subject getSubject(Context context) {
        return new Subject.SubjectBuilder()
            .context(context)
            .build();
    }
}
```

[Back to top](#top)

<a name="events" />

## 2. Tracking Events

Once you have successfully built your Tracker object you can track events with calls like the following:

```java
Tracker tracker = getTracker(activity, context);
tracker.track(ScreenView.builder().name("screenName").id("screenId").build());
```

For an outline of all available tracking combinations have a look [here][demo-app-track-events].

[Back to top](#top)

<a name="app-focus" />

## 3. Application Focus

The Tracker Session object can be tuned to timeout in `foreground` and `background` scenarios, but you are required to tell us when your application is in these states.  Unfortunately it is not possible to do so from a library standpoint.

For Android APIs lower than 14 the current implementation we are using is to override the `onPause()` and `onResume()` functions of an application activity to notify the session when we change states.

```java
@Override
protected void onPause() {
    super.onPause();
    tracker.getSession().setIsBackground(true);
}

@Override
protected void onResume() {
    super.onResume();
    tracker.getSession().setIsBackground(false);
}
```

For Android APIs 14+ we utilise a Lifecycle Handler class to manage this for us.  Simply setup the handler with your application activity like so:

```java
Tracker.instance().setLifecycleHandler({{ APPLICATION_ACTIVITY }})
```

[Back to top](#top)

[android-tracker]: https://github.com/snowplow/snowplow/wiki/Android-Tracker
[setup-guide]: https://github.com/snowplow/snowplow/wiki/Android-Tracker-Setup
[demo-app-track-events]: https://raw.githubusercontent.com/snowplow/snowplow-android-tracker/master/snowplow-demo-app/src/main/java/com/snowplowanalytics/snowplowtrackerdemo/utils/TrackerEvents.java
[0.4.0]: https://github.com/snowplow/snowplow/wiki/Android-Integration-0.4.0
[0.5.0]: https://github.com/snowplow/snowplow/wiki/Android-Integration-0.5.0
