<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**Node.js tracker**](Nodejs-tracker-setup)

## Contents

- 1 [Overview](#overview)
- 2 [Compatibility](#compatibility)
- 3 [Installation](#installation)  
- 4 [Next steps](#next-steps)

<a name="overview" />

## 1. Overview

The [Snowplow Node.js Tracker](https://github.com/snowplow/snowplow-nodejs-tracker) lets you add analytics to your [Node.js][node-js]-based applications.

<a name="compatibility" />

## 2. Compatibility

The Snowplow Node.js Tracker is compatible with Node.js versions 0.10.x and 0.11.x. Installing it requires [npm][npm].

<a name="installation" />

## 3. Installation

Setting up the tracker should be straightforward if you are familiar with npm:

```bash
npm install snowplow-tracker
```

You can automatically save the Tracker as a dependency for your npm module like this:

```bash
npm install snowplow-tracker --save
```

<a name="next-steps" />

## 4. Next steps

Once you've installed the module, check out the [technical documentation page](Node.js-Tracker) for information on the Node.js Tracker API. Then you should be ready to start tracking your own events.

[Back to top](#top)

[node-js]: http://nodejs.org/
[npm]: https://www.npmjs.org/
