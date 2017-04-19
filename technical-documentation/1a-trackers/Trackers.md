[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » Trackers

[[https://d3i6fms1cm1j0i.cloudfront.net/github-wiki/images/snowplow-architecture-1a-trackers.png]]

## Overview

**Trackers** are client- or server-side libraries which track customer behaviour by sending Snowplow events to a [Collector](collectors).

## Tracker documentation

### Trackers

* [[ActionScript3 Tracker]] - for tracking events in your Flash Player 9+, Flash Lite 4 and AIR games, apps and widgets
* [[Arduino Tracker]] - for tracking events from an IP-connected Arduino board
* [[Android Tracker]] - for tracking events from your Android-based applications, games or frameworks
* [[CPP Tracker]] - for tracking events from your C++ apps, games and servers
* [[Golang Tracker]] - for tracking events from your Golang apps and servers
* [[Google AMP Tracker]] - is natively integrated into the AMP project
* [[iOS Tracker]] - for tracking events in your iOS apps and games
* [[Java Tracker]] - for tracking events in your Java-based desktop and server apps, servlets and games
* [JavaScript Tracker](Javascript-Tracker) - for tracking user activity on websites and webapps
* [[Lua Tracker]] - track events in your Lua-based applications, Lua web servers/frameworks, or from the Lua scripting layer within your games or apps
* [[.NET Tracker]] - for tracking events in your .NET applications
* [[Node.js Tracker]] - track events from Node.js applications
* [[PHP Tracker]] - track events from your PHP based applications, games and frameworks
* [[Python Tracker]] - for tracking events in your Python-based applications, games or Python web servers/frameworks
* [Pixel Tracker](pixel-tracker) - a pixel tracker for web environments where JavaScript is not available
* [[Ruby Tracker]] - for tracking events in your Ruby and Rails apps and gems
* [[Scala Tracker]] - for tracking events in your Scala apps and servers
* [[Unity Tracker]] - for tracking events from your Unity games and apps

For other trackers (e.g. Erlang, C++) and their approximate timelines, please see the [Product roadmap](Product-roadmap).

### Protocol

* [Snowplow Tracker Protocol](snowplow-tracker-protocol) - the protocol implemented by all trackers
