<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: Setup a Tracker**](Setting-up-a-Tracker) > [**Objective-C tracker**](ios-tracker-setup)

This page refers to version 0.6.2 of the Snowplow Objective-C Tracker, which is the latest version. Documentation for earlier versions is available:

* *[Version 0.5][ios-0.5]*
* *[Version 0.4][ios-0.4]*
* *[Version 0.1-0.3][ios-0.3]*

## Contents

- 1. [Overview](#overview)
- 2. [Integration options](#integration-options)
  - 2.1 [Tracker compatibility](#compatibility)  
  - 2.2 [Dependencies](#dependencies)
- 3. [Setup](#setup)
  - 3.1 [CocoaPods](#cocoapods)
  - 3.2 [Manual setup](#manual-setup)
    - 3.2.1 [FMDB Setup](#fmdb-setup)
    - 3.2.2 [Reachability Setup](#reach-setup)
    - 3.2.3 [Import all required frameworks](#required-frameworks)
  - 3.3 [Static setup](#static-setup)
    - 3.3.1 [OpenIDFA Setup](#open-idfa-setup)

<a name="overview" />
## 1. Overvew

The [Snowplow Objective-C Tracker](https://github.com/snowplow/snowplow-objc-tracker) lets you add analytics to your [iOS] [ios], [OSX][osx] and [tvOS][tvos] applications.

The Tracker should be relatively straightforward to setup if you are familiar with iOS/OSX development using CocoaPods.

Ready? Let's get started.

[Back to top](#top)

<a name="integration-options" />
## 2. Integration options

<a name="compatibility" />
### 2.1 Tracker compatibility

With iOS backward compatibility is limited to a small range which it makes it easy to support. Hence, our goal was to make the iOS Tracker compatible with iOS 7+ with easy availability via CocoaPods.

[Back to top](#top)

<a name="dependencies" />
### 2.2 Dependencies

* For iOS the Tracker has dependencies limited to the [FMDB][fmdb] and [Reachability][reach] libraries for database management and network information respectively. Both of which have iOS 7+ support as well.

* For OSX and tvOS the Tracker has dependencies limited to [FMDB][fmdb] for database management.

If you're installing via CocoaPods, the dependencies are recursively downloaded for your project so you don't have to worry about it.

[Back to top](#top)

<a name="setup" />
## 3. Setup

<a name="cocoapods" />
### 3.1 CocoaPods

We support installing the Objective-C Tracker via CocoaPods since it's the easiest way to install the tracker. Doing so is simple:

1. Install CocoaPods using `gem install cocoapods`
2. Create the file `Podfile` in the root of your XCode project directory, if you don't have one already
3. Add the following line into it:
```
pod 'SnowplowTracker'
```
4. Run `pod install` in the same directory

[Back to top](#top)

<a name="manual-setup" />
### 3.2 Manual Setup

If you prefer not to use CocoaPods, you can grab the tracker from our [GitHub repo][objc-tracker-github] and import it into your project.

#### Clone the tracker

First, git clone the latest version of the tracker to your local machine:

    git clone https://github.com/snowplow/snowplow-objc-tracker.git

If you don't have git installed locally, [install it] [git] first.

#### Copy the tracker into your project

You only need to copy the tracker's `Snowplow` sub-folder into your XCode project's folder. The command will look something like this:

    cp -r snowplow-objc-tracker/Snowplow MyObjcApp/MyObjcApp/

Replace `MyObjcApp` with the name of your own app, and tweak the source code sub-folder accordingly.

Next, drag and drop the sub-folder `MyObjcApp/MyObjcApp/Snowplow` into your XCode project's workspace:

[[/setup-guide/images/setup-objc-tracker-manual-1.png]]

Make sure that the suggested options for adding `Snowplow` are set **Create groups**, then click **Finish**:

[[/setup-guide/images/setup-objc-tracker-manual-2.png]]

<a name="fmdb-setup" />
#### 3.2.1 Add the FMDB dependency

The tracker is dependent on [FMDB] [fmdb] version 2.3, an Objective-C wrapper around SQLite.

As before, git clone the dependency and copy the source into your XCode project's folder:

```
git clone https://github.com/ccgus/fmdb.git
cd fmdb && git checkout v2.3
cp -r src/fmdb ../MyObjcApp/MyObjcApp/
```

As before, drag and drop the sub-folder `MyObjcApp/MyObjcApp/fmdb` into your XCode project's workspace, making sure to **Create groups**.

Finally, you will need to edit `Snowplow/SPEmitter.m` and `Snowplow/SPEventStore.m` in X-Code and change:

```objective-c
#import <FMDB.h>
```

to:

```objective-c
#import "FMDB.h"
```

[Back to top](#top)

<a name="reach-setup" />
#### 3.2.2 Add the Reachability dependency

For iOS only, the tracker is dependent on [Reachability] [reach] version 3.2, a drop in replacement for Apple's Reachability class.

As before, git clone the dependency and copy the source into your XCode project's folder:

```
git clone https://github.com/tonymillion/Reachability.git
cd Reachability && git checkout v3.2
cp Reachability.{h,m} ../MyObjcApp/MyObjcApp/
```

Now add the `Reachability.{h,m}` files to your project by:

* Right-clicking on your `MyObjcApp` folder in XCode
* Selecting Add Files to "MyObjcApp"...
* Selecting both Reachability files and adding them

Once you have added the `.h/m` files to the project:

* Go to `Projects->TARGETS->Build Phases->Link Binary With Libraries`
* Press the plus in the lower left of the list
* Add `SystemConfiguration.framework`

[Back to top](#top)

<a name="required-frameworks" />
#### 3.2.3 Import all required frameworks

The tracker also depends on various frameworks:

```
$ grep 'frameworks' ../snowplow-objc-tracker/SnowplowTracker.podspec
  s.ios.frameworks = 'CoreTelephony', 'UIKit', 'Foundation'
  s.osx.frameworks = 'AppKit', 'Foundation'
```

Go to **Target** > **General** tab > **Linked Frameworks and Libraries** section and add:

1. All of the frameworks for your target platform, as returned by the `grep` above
2. `libsqlite3.dylib`

This is what adding the frameworks to an OS-X application looks like:

[[/setup-guide/images/setup-objc-tracker-manual-3.png]]

#### Building

Now **Build** your project and you should see **Build Succeeded**. If you get a build error, check that you added Snowplow and fmdb as Groups, not just as Folders.

You are now ready to proceed to instrumenting your app. Just remember to use quotation marks not angle brackets, and the `Snowplow` sub-folder as necessary when importing the tracker:

```objective-c
#import "SPTracker.h"
#import "SPEmitter.h"
```

[Back to top](#top)

<a name="static-setup" />
### 3.3 Static Library Setup

**Currently the Static Library is only available for iOS, not for OS-X.**

We also now support including the SnowplowTracker as a static library import.

* Download the static library's zipfile from [bintray][static-download]
* Unzip the static library's zipfile to a location (doesn't need to be inside your app)
* Go to `Projects->TARGETS->Build Phases->Link Binary With Libraries`
* Press the plus in the lower left of the list
* Navigate to where you downloaded the Tracker to and then add the framework

To activate the library:

* Go to `Projects->TARGETS->Build Settings`
* Make sure you are browsing `All` settings, not just `Basic`
* Search for `Other Linker Flags`
* Ensure that the `-ObjC` flag has been added

To import the library headers into your project you will need to add them with the name of the library prepended like so:

```objective-c
#import "SnowplowTracker/SPTracker.h"
```

This portion was based on the developers guide from Apple on [importing static libraries][apple-static-instructions].

*Please note* that you will need to add in all Tracker dependencies manually as they are not included in the static download.

* To add FMDB: [FMDB Setup](#fmdb-setup)
* To add Reachability: [Reachability Setup](#reach-setup)
* To add the required frameworks: [Import all required frameworks](#required-frameworks)

[Back to top](#top)

<a name="open-idfa-setup" />
#### 3.3.1 Add the OpenIDFA dependency

If you are embedding this app in an iOS app, then you will need to add [OpenIDFA] [openidfa], which has been intentionally excluded from the Static Library.

**Do not add OpenIDFA for a OS-X app.**

As before, git clone the dependency and copy the source into your XCode project's folder:

```
git clone https://github.com/ylechelle/OpenIDFA.git
git checkout 382db601ca12a7710b8fe583e710f9cfcbf42c4f
cd OpenIDFA
cp OpenIDFA.{h,m} ../MyObjcApp/MyObjcApp/
```

Now add the `OpenIDFA.{h,m}` files to your project by:

* Right-clicking on your `MyObjcApp` folder in XCode
* Selecting Add Files to "MyObjcApp"...
* Selecting both OpenIDFA files and adding them

[Back to top](#top)

[ios]: https://developer.apple.com/technologies/ios/
[osx]: https://www.apple.com/osx/
[tvos]: https://www.apple.com/tv/
[objc-tracker-github]: https://github.com/snowplow/snowplow-objc-tracker

[static-download]: https://bintray.com/artifact/download/snowplow/snowplow-generic/snowplow_objc_tracker_0.6.2.zip
[apple-static-instructions]: https://developer.apple.com/library/ios/technotes/iOSStaticLibraries/Articles/configuration.html

[git]: http://git-scm.com/downloads
[fmdb]: https://github.com/ccgus/fmdb
[reach]: https://github.com/tonymillion/Reachability
[openidfa]: https://github.com/ylechelle/OpenIDFA

[ios-0.3]: https://github.com/snowplow/snowplow/wiki/iOS-Tracker-Setup-0.1-0.3
[ios-0.4]: https://github.com/snowplow/snowplow/wiki/iOS-Tracker-Setup-0.4
[ios-0.5]: https://github.com/snowplow/snowplow/wiki/iOS-Tracker-Setup-0.5
