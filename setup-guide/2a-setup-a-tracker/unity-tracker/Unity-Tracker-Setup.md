<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**Unity Tracker**](Unity-Tracker-Setup)

## Contents

- 1 [Overview](#overview)  
- 2 [Integration options](#integration-options)
  - 2.1 [Tracker compatibility](#compatibility)  
  - 2.2 [Dependencies](#dependencies)
- 3 [Setup](#setup)
  - 3.1 [Download](#download)

<a name="overview" />

## 1. Overview

The [Snowplow Unity Tracker](https://github.com/snowplow/snowplow-unity-tracker) lets you add analytics to your [Unity][unity]-based desktop and mobile apps, servlets and games.

The Tracker should be relatively straightforward to setup if you are familiar with Unity and C# development.

Ready? Let's get started.

[Back to top](#top)

<a name="integration-options" />

## 2. Integration options

<a name="compatibility" />

### 2.1 Tracker compatibility

The Snowplow Unity Tracker has been built and tested using Unity5+ on Windows, Linux, OSX and iOS.

It is not currently compatible with the WebPlayer as of yet.

[Back to top](#top)

<a name="dependencies" />

### 2.2 Dependencies

There are several dependencies that are required to use the Unity Tracker currently.  These are available in [this folder][deps-folder].

* `UnityEngine.dll` : This is included as a way of setting up development externally to the Unity playground; you do not need to include this.

All the other dependencies are required for proper Tracker function.

[Back to top](#top)

<a name="setup" />

## 3. Setup

<a name="download" />

### 3.1 Download

The Unity Tracker package is published to Snowplow's [bintray repository][bintray-snplow].  Once downloaded simply add the `SnowplowTracker.unitypackage` to your project.  This should insert all of the required dll files into your `Assets/Plugins/` folder.

Done? Now read the [Unity Tracker API](Unity-Tracker) to start tracking events.

[unity]: https://unity3d.com/
[deps-folder]: https://github.com/snowplow/snowplow-unity-tracker/tree/master/Resources/Assets/Plugins
[unity-http-home]: https://github.com/snowplow/snowplow-unity-tracker/tree/master/UnityHTTP
[unity-http]: https://github.com/andyburke/UnityHTTP/tree/master/src
[unity-json-home]: https://github.com/snowplow/snowplow-unity-tracker/tree/master/UnityJSON
[bintray-snplow]: http://dl.bintray.com/snowplow/snowplow-generic/snowplow_unity_tracker_0.1.0.zip
