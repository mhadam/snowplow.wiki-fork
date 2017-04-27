<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**JavaScript tracker core**](Javascript-tracker-core-setup)

## Contents

- 1. [Overview](#overview)
- 2. [Compatibility](#compatibility)
- 3. [Installation](#installation)
- 4. [Next steps](#next-steps)

<a name="overview" />

## 1. Overview

The [Snowplow JavaScript Tracker Core](https://github.com/snowplow/snowplow-javascript-tracker/tree/master/core) is a JavaScript library for Snowplow trackers. It is currently used by the Node.js Tracker, and will soon be used by the client-side JavaScript Tracker.

<a name="compatibility" />

## 2. Compatibility

The Tracker Core is compatible with [Node.js][node-js] versions 0.10.x and 0.11.x. Installing it requires [npm][npm].

<a name="installation" />

## 3. Installation

Setting up the tracker should be straightforward if you are familiar with npm:

```bash
npm install snowplow-tracker-core
```

You can automatically save the Tracker Core as a dependency for your npm module like this:

```bash
npm install snowplow-tracker-core --save
```

<a name="next-steps" />

## 4. Next steps

Once you've installed the module, look at the [technical documentation page](Javascript-Tracker-Core) for information on the JavaScript Tracker Core API.

[Back to top](#top)

[node-js]: http://nodejs.org/
[npm]: https://www.npmjs.org/
