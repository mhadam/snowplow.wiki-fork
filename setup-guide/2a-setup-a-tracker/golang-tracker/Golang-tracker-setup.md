<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**Golang Tracker**](Golang-tracker-setup)

## Contents

- 1. [Overview](#overview)  
- 2. [Tracker compatibility](#compatibility)  
- 3. [Setup](#setup)

<a name="overview" />

## 1. Overview

The [Snowplow Golang Tracker](https://github.com/snowplow/snowplow-golang-tracker) lets you add analytics to your [Golang][golang]-based apps and servers.

Ready? Let's get started.

[Back to top](#top)

<a name="compatibility" />

## 2 Tracker compatibility

The Snowplow Golang Tracker has been built and tested using Golang versions 1.3, 1.4, 1.5 and 1.6.

[Back to top](#top)

<a name="setup" />

## 3. Setup

The Tracker is hosted on Github and versions of the Tracker are fetched using [gopkg](http://labix.org/gopkg.in).

To get the package, execute:

```bash
$host go get gopkg.in/snowplow/snowplow-golang-tracker.v1/tracker
```

To import the package, add the following line to your code:

```go
import "gopkg.in/snowplow/snowplow-golang-tracker.v1/tracker"
```

The current version of the Snowplow Golang Tracker is 1.1.0.

[Back to top](#top)

Done? Now read the [Golang Tracker API](Golang-Tracker) to start tracking events.

[golang]: https://golang.org/
