<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**CPP Tracker**](CPP-tracker-setup)

## Contents

- 1. [Overview](#overview)  
- 2. [Tracker compatibility](#compatibility)  
- 3. [Setup](#setup)

<a name="overview" />

## 1. Overview

The [Snowplow C++ Tracker](https://github.com/snowplow/snowplow-cpp-tracker) lets you add analytics to your [C++][cpp] based apps, games and servers.

Ready? Let's get started.

[Back to top](#top)

<a name="compatibility" />

## 2 Tracker compatibility

The Snowplow C++ Tracker has been built and tested using C++11 as a minimum.  

Supported operating systems:

- OSX
- Windows

[Back to top](#top)

<a name="setup" />

## 3. Setup

The Tracker is hosted on Github and versions of the Tracker will need to be included by manually dragging and dropping the source files directly into your codebase.

Everything in both the `src` and `include` folders will need to be included in your application.  It is important to keep the same folder structure as references to the included headers have been done like so: `../include/json.hpp`.

If you wish to change this you will need to manually update the `src` header files.

__NOTE__: We have included the raw `sqlite3.h & c` files which will need to be compiled seperately before being linked to your main C++ project. 

The current version of the Snowplow C++ Tracker is 0.1.0.

[Back to top](#top)

Done? Now read the [C++ Tracker API](CPP-Tracker) to start tracking events.

[cpp]: http://www.cplusplus.com/info/description/
