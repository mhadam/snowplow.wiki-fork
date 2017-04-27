<a name="top" />

This page is a walkthrough detailing how the demo app works, what it is doing and how to use it.  As well as code samples to get you started on integration!

For previous versions:

* [v0.2.0][0.2.0]
* [v0.1.0][0.1.0]

## Contents

- 1. [Overview](#overview)
- 2. [App Walkthrough](#walkthrough)
- 3. [Code Review](#review)

<a name="overview" />

## 1. Overview

Before we get started please [download the APK][download-apk] and install it on your Android device.  You will need to allow installation from [unknown sources][unknown-sources] before it will proceed.

The application has been developed as a proof of concept of the Android Tracker in all of its forms.  It will allow you to send all combinations of possible events to an endpoint of your choosing so you can see how the events look and what data is being collected.

For help in setting up a local endpoint please go [here][testing].

[Back to top](#top)

<a name="walkthrough" />

## 2. App Walkthrough

The main screen:

[[/technical-documentation/images/android/android-app-1-0.3.0.png]]

From this screen you will be able to select the Tracker Demo screen.

[[/technical-documentation/images/android/android-app-2-0.3.0.png]]

Once on this page the Tracker has been created and you will notice a few options to set as well as some metrics which will update as you run the demo.

Metrics Explained:

- Made: The count of events created since you started the demo.
- Sent: The count of successfully sent events since you started the demo.
- Online: Whether or not the Tracker has access to the internet (turn on Aeroplane mode to see this switch).
- Running: Whether or not the Emitter is actively trying to send.
- DB Size: How many events are currently stored in the applications database.
- Sessions #: The amount of sessions that have occured during this apps lifetime.

Options Explained:

- Emitter URI: This is where you will need to fill in a valid endpoint to send events to.
- Emitter Protocol: Whether to use HTTP or HTTPS for event sending.
- Emitter Type: Whether to send events as GET's or POST's.
  - You will note quite a substantial speed increase if using POST over GET.
- Collection: Whether to turn event collection ON or OFF.

So now that we know what everything is lets start the demo!

So we have entered in a valid URI for the application and we are sending events to it in POST's.

Everytime the emitter successfully sends off a batch a log will appear in the Emitter Callback section of the demo, this will denote the Successes (or Failures) that occured during sending.  As events are sent you will also notice that the DB Size will gradually go back to 0.

[[/technical-documentation/images/android/android-app-3-0.3.0.png]]

In the last frame you will now notice that we have successfully sent all of our events to the endpoint, the DB Size has gone back to 0 and the emitter will soon stop searching for events to send.

[Back to top](#top)

<a name="review" />

## 3. Code Review

Now that we have seen the application in action we can walkthrough what's happening and work on how to integrate it into your codebase.  For some example classes please have a look at the [integration guide][integration].  In the integration guide you will see some very bare-bones implementations of both the Classic and RxJava Trackers, these both use all of the default settings however you are encouraged to test the settings until you have something that works for you!

One thing to note here is that all aspects, apart from initial creation, of the Tracker are performed in background threads.  So you do not need to worry about running a `tracker.trackXXX` function from your UI/Main Thread, the Tracker should never block the main thread.  As you can see [here][calling-from-ui] the demo app calls all of the track functions whilst on the UI Thread with zero impact on the actual running of the UI.

This [file][events-examples] contains many combinations of events that can be sent by the tracker filled with dummy information.  These dummy events are what gets sent to the specified endpoint.  As well as an example of how to build a [custom context][custom-context] to add further decoration to your events.

This should give you a fair amount of examples to get started!  If you need any help please do not hesitate to post questions in the [google user group][user-group].

[Back to top](#top)

[download-apk]: http://dl.bintray.com/snowplow/snowplow-generic/snowplow-demo-app-release-0.3.0.apk
[unknown-sources]: http://developer.android.com/distribute/tools/open-distribution.html
[integration]: https://github.com/snowplow/snowplow/wiki/Android-Integration
[testing]: https://github.com/snowplow/snowplow/wiki/Android-Testing-locally-and-Debugging
[calling-from-ui]: https://github.com/snowplow/snowplow-android-tracker/blob/master/snowplow-demo-app/src/main/java/com/snowplowanalytics/snowplowtrackerdemo/Demo.java#L114
[events-examples]: https://github.com/snowplow/snowplow-android-tracker/blob/master/snowplow-demo-app/src/main/java/com/snowplowanalytics/snowplowtrackerdemo/utils/TrackerEvents.java#L41-L87
[custom-context]: https://github.com/snowplow/snowplow-android-tracker/blob/master/snowplow-demo-app/src/main/java/com/snowplowanalytics/snowplowtrackerdemo/utils/TrackerEvents.java#L92-L100
[user-group]: https://groups.google.com/forum/#!forum/snowplow-user
[0.1.0]: https://github.com/snowplow/snowplow/wiki/Android-app-walkthrough-0.1.0
[0.2.0]: https://github.com/snowplow/snowplow/wiki/Android-app-walkthrough-0.2.0
