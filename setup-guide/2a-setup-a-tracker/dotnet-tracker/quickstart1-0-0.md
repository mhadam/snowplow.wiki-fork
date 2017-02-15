# .NET Tracker Quickstart

### With Xamarin

#### Install the NuGet package

The package is available on NuGet, you can use the Visual Studio package manager as follows:

`Package-Install Snowplow.Tracker.PlatformExtensions`

Alternatively using the Visual Studio GUI to install the package `Snowplow.Tracker.PlatformExtensions`. You don't need to include the `Snowplow.Tracker` package if you're using the `Snowplow.Tracker.PlatformExtensions` package,
it's implicitly included for you. 

#### Instantiating the tracker

If you're using a Xamarin PCL project - you should include the tracker into the (shared) PCL project. The tracker will then be available for your iOS and Android projects too.
You'll need to instantiate the tracker before you use it, the tracker follows a singleton pattern. When you start your application, you should start the tracker - and when your application is closing
it's important to close the tracker too.

```{csharp}         
// This is a provided class that sends logs to STDOUT - but you can implement the interface and route the tracker logs anywhere
var logger = new ConsoleLogger();

// This is where your Snowplow collector lives
var dest = new SnowplowHttpCollectorEndpoint(emitterUri, method: method, port: port, protocol: protocol, l: logger);

// Note: Maintain reference to Storage as this will need to be disposed of manually
_storage = new LiteDBStorage(SnowplowTrackerPlatformExtension.Current.GetLocalFilePath("events.db"));
var queue = new PersistentBlockingQueue(_storage, new PayloadToJsonString());

// Note: When using GET requests the sendLimit equals the number of concurrent requests - to many of these will break your application!
var sendLimit = method == HttpMethod.GET ? 10 : 100;

// Note: To make the tracker more battery friendly and less likely to drain batteries there are two settings to take note of here:
//       1. The stopPollIntervalMs: Controls how often we look to the database for more events
//       2. The deviceOnlineMethod: Is run before going to the database or attempting to send events, this will prevent any I/O from
//          occurring unless you have an active network connection
var emitter = new AsyncEmitter(dest, queue, sendLimit: sendLimit, stopPollIntervalMs: 1000, sendSuccessMethod: EventSuccessCallback, 
    deviceOnlineMethod: SnowplowTrackerPlatformExtension.Current.IsDeviceOnline, l: logger);

var userId = PropertyManager.GetStringValue(KEY_USER_ID, SnowplowCore.Utils.GetGUID());
PropertyManager.SaveKeyValue(KEY_USER_ID, userId);

var subject = new Subject()
    .SetPlatform(Platform.Mob)
    .SetUserId(userId)
    .SetLang("en");

if (useClientSession)
{
    _clientSession = new ClientSession(SnowplowTrackerPlatformExtension.Current.GetLocalFilePath("client_session.dict"), l: logger);
}

// Note: You can either attach contexts to each event individually or for the more common contexts such as Desktop, Mobile and GeoLocation
//       you can pass a delegate method which will then be called for each event automatically.

MobileContextDelegate mobileContextDelegate = null;
if (useMobileContext)
{
    mobileContextDelegate = SnowplowTrackerPlatformExtension.Current.GetMobileContext;
}

GeoLocationContextDelegate geoLocationContextDelegate = null;
if (useMobileContext)
{
    geoLocationContextDelegate = SnowplowTrackerPlatformExtension.Current.GetGeoLocationContext;
}

// Attach the created objects and begin all required background threads!
Instance.Start(emitter: emitter, subject: subject, clientSession: _clientSession, trackerNamespace: _trackerNamespace, 
    appId: _appId, encodeBase64: false, synchronous: false, mobileContextDelegate: mobileContextDelegate, 
    geoLocationContextDelegate: geoLocationContextDelegate, l: logger);
```
  

To send events:

```{csharp}     
// This example is to send a Screen View event
Instance.Track(new ScreenView()
    .SetId("example-screen-id")
    .SetName("Example Screen")
    .Build());
```

To stop the tracker:


```{csharp}     
// Note: This will also stop the ClientSession and Emitter objects for you!
Instance.Stop();

// Note: Dispose of Storage to remove lock on database file!
if (_storage != null)
{
    _storage.Dispose();
    _storage = null;
}

if (_clientSession != null)
{
    _clientSession = null;
}

SnowplowTrackerPlatformExtension.Current.StopLocationUpdates();
```

We've also created a detailed sample mobile application for you [mobile-app-demo][mobile-demo].

### With .NET Framework 4.6.1+ and Snowplow.Tracker.PlatformExtensions

The platform extensions library, in addition to Xamarin provides a .NET 4.6.1 implementation that allows use of MSMQ as a storage solution (the default is still LiteDB - MSMQ is optional). It also contains information on if the device is online.

As before, to start the tracker:

```{csharp}     
// This is a provided class that sends logs to STDOUT - but you can implement the interface and route the tracker logs anywhere
var logger = new ConsoleLogger();

// This is where your Snowplow collector lives
var dest = new SnowplowHttpCollectorEndpoint(emitterUri, method: method, port: port, protocol: protocol, l: logger);

// Note: Maintain reference to Storage as this will need to be disposed of manually
_storage = new LiteDBStorage(SnowplowTrackerPlatformExtension.Current.GetLocalFilePath("events.db"));
var queue = new PersistentBlockingQueue(_storage, new PayloadToJsonString());

// Note: When using GET requests the sendLimit equals the number of concurrent requests - to many of these will break your application!
var sendLimit = method == HttpMethod.GET ? 10 : 100;

// Note: To make the tracker more battery friendly and less likely to drain batteries there are two settings to take note of here:
//       1. The stopPollIntervalMs: Controls how often we look to the database for more events
//       2. The deviceOnlineMethod: Is run before going to the database or attempting to send events, this will prevent any I/O from
//          occurring unless you have an active network connection
var emitter = new AsyncEmitter(dest, queue, sendLimit: sendLimit, stopPollIntervalMs: 1000, sendSuccessMethod: EventSuccessCallback, 
    deviceOnlineMethod: SnowplowTrackerPlatformExtension.Current.IsDeviceOnline, l: logger);

// Attach the created objects and begin all required background threads!
Instance.Start(emitter: emitter, subject: subject, clientSession: _clientSession, trackerNamespace: _trackerNamespace, 
    appId: _appId, encodeBase64: false, synchronous: false, mobileContextDelegate: mobileContextDelegate, 
    geoLocationContextDelegate: geoLocationContextDelegate, l: logger);
```

To send events:

```{csharp}     
// This example is to send a Page View event
Instance.Track(new PageView()
    .SetPageUrl("http://example.page.com")
    .SetReferrer("http://example.referrer.com")
    .SetPageTitle("Example Page")
    .Build());
```
And finally, it's important to stop the tracker when your application is closing:

```{csharp}     
Instance.Stop();

// Note: Dispose of Storage to remove lock on database file!
if (_storage != null)
{
    _storage.Dispose();
    _storage = null;
}
```

#### Install the NuGet package

Using the Visual Studio package manager as follows:

`Package-Install Snowplow.Tracker`

#### Creating and using the tracker

The API when using the core library is slightly different to those using the PlatformExtensions.

To create an instance of the tracker:

```{csharp}     
var logger = new ConsoleLogger();
Tracker.Tracker.Instance.Start(collectorHostname, "snowplow-demo-app.db", l: logger);
```

To send events:

```{csharp}     
// This example sends a PageView event
Tracker.Tracker.Instance.Track(new PageView().SetPageUrl("http://helloworld.com/sample/sample.php").Build());
```

And finally, as with the platform extensions it's important to close the tracker as your application exits.

```{csharp}     
// Flushing is optional - it'll try sending all the events that haven't been sent yet
Tracker.Tracker.Instance.Flush();
Tracker.Tracker.Instance.Stop();
```

In addition to the Xamarin sample project, we have a comparable one for .NET core [here][demo-core]. 

[mobile-demo]: https://github.com/snowplow/snowplow-dotnet-tracker/tree/release/1.0.0/Snowplow.Demo.App/Snowplow.Demo.App
[demo-core]: https://github.com/snowplow/snowplow-dotnet-tracker/blob/release/1.0.0/Snowplow.Demo.Console/Program.cs