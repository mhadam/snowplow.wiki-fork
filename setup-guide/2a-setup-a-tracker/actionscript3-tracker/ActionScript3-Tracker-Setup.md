<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**Actionscript 3 Tracker**](as3-tracker-setup)

## Contents

- 1 [Overview](#overview)  
- 2 [Integration options](#integration-options)
  - 2.1 [Tracker compatibility](#compatibility)  
  - 2.2 [Dependencies](#dependencies)
- 3 [Setup](#setup)
  - 3.1 [Hosting](#hosting)
  - 3.2 [FlashBuilder](#flashbuilder)

<a name="overview" />

## 1. Overview

The [Snowplow Actionscript 3 (AS3) Tracker][github-snowplow-as3] allows you to track Snowplow events from a Flash movie, Flex application or Adobe AIR application.

The Tracker is straightforward to setup.

Ready? Let's get started.

[Back to top](#top)

<a name="integration-options" />

## 2. Integration options

<a name="compatibility" />

### 2.1 Tracker compatibility

The Snowplow AS3 Tracker has been built, tested and compiled using the Adobe Flex 3.5 SDK, but is not dependent on Flex. You can use it in pure Actionscript projects.

It is compatible with Flash Player 9.0.124 and later.

[Back to top](#top)

<a name="dependencies" />

### 2.2 Dependencies

There are no external dependencies. The FlashBuilder project includes any necessary external classes, and the binary swc file is standalone, ready to be included in your project and referenced in your AS3 code.

[Back to top](#top)

<a name="setup" />

## 3. Setup

<a name="hosting" />

### 3.1 Hosting

The Tracker is available as source code for inclusion and compilation in your project, or as a binary .SWC file which you can just add to your project as is.

The binary is published to Snowplow's [hosted Bintray repository][bintray-snowplow-as3].

The source code is available on [Snowplow's GitHub repository][github-snowplow-as3].

The current version of the Snowplow AS3 Tracker is 0.1.0.

<a name="flashbuilder" />

### 3.2 FlashBuilder

If you are using FlashBuilder for building your Flash application, then either:

#### 3.2.1 Install using binary

The binary is available for download from our public BinTray repository. Installation steps as follows:

1. Download the [snowplow_actionscript3_tracker_0.1.0.zip][bintray-snowplow-as3] file
2. Unzip the binary (.SWC) file
3. Include it in your project under "Properties > Build Path > Library Path > Add SWC..."

#### 3.2.2 Install from source

You can also install this Flash analytics SDK from source:

1. Download the project source code from GitHub under the [0.1.0 tag][source-snowplow-as3]
2. Import the source into your FlashBuilder project using "Import > Other... > Existing Projects into Workspace"
3. Link this new project to your project using "Properties > Build Path > Library Path > Add Project..."

Done? Now read the [Actionscript 3 Tracker API](./ActionScript3-Tracker) to start tracking events.

[flash]: http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/
[github-snowplow-as3]: https://github.com/snowplow/snowplow-actionscript3-tracker
[bintray-snowplow-as3]: http://dl.bintray.com/snowplow/snowplow-generic/snowplow_actionscript3_tracker_0.1.0.zip
[source-snowplow-as3]: https://github.com/snowplow/snowplow-actionscript3-tracker/tree/0.1.0
