<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow technical documentation) » [**Trackers**](trackers) » [**JavaScript Tracker**](Javascript-Tracker) » For Developers

**THIS DOCUMENT IS WIP**

*This page refers to version 2.7.0 of the Snowplow JavaScript Tracker*  

<a name="for-developers" />
## 5. For developers
  - 5.1 [Tools](#tools)
  - 5.2 [Tag](#tag)
  - 5.3 [Tracker modules](#modules)
    - 5.3.1 [js/init.js](#init)
    - 5.3.2 [js/snowplow.js](#snowplow)
    - 5.3.3 [js/in_queue.js](#in_queue)
    - 5.3.4 [js/tracker.js](#tracker)
    - 5.3.5 [core/lib/core.ts](#core)
    - 5.3.5 [js/out_queue.js](#out_queue)
  - 5.4 [Tests](#tests)
    - 5.4.1 [Automated testing](#autotest)
    - 5.4.2 [Manual testing](#manualtest)
  - 5.5 [Build](#build)

<a name="tools" />
## 5.1 Tools

In JavaScript tracker development we're using following tools:

* [Grunt][grunt] - build system
* [Browserify][browserify] - tool allowing to write [CommonJS][commonjs] modules and run them in browser afterwards
* [Npm & co][npmjs] - JavaScript package manager; all other dependencies can be found in `package.json`
* [TypeScript][typescript] - JavaScript transpiller with static types
* [Intern][intern] - test framework
* [Sauce Labs][saucelabs] - Cross-browser automated testing platofrm (functional tests only)
* [Ngrok][ngrok] - localhost tunneling solution (integration tests suite only)

None of above tools are essential for JavaScript tracker, but they are necessary to build and test it.

<a name="tag" />
## 5.2 Tag

As all programs, JavaScript tracker has an entry point from where tracker gets 
initialized. In our case this is a tag, which website owner need to insert into 
pages where JavaScript tracker should be loaded.
This `script` tag used to be minified as much as possible and at first glance
may seem cryptic, but you can find expanded version with all explanations in
`tags/tag.js` file. This tag attaches `<script>` with *actual tracker code* to
`document` and then initalizes snowplow namespace object as global function that
simply takes all arguments and pushes them to `q` object inside itself.

So, essentially primary tracker object (usually called "namespace object", by
default - `snowplow_name_here`) is `Function` object with `q` property attached
to it. While execution still in the tag - `q` is simple JS `Array`.

<a name="modules" />
## 5.3 Tracker modules

<a name="init" />
### 5.3.1 js/init.js

When *actual tracker code* loads - we're getting into second entry point - 
`js/init.js`. Here `q` becomes `snowplow.Snowplow` object (`InQueueManager` 
more precisely) contating what `q` array was containing before (browser could 
push many events there since script loading is asynchronous). You can ignore 
everything with `_snaq` - this is legacy queue which will be removed soon.

<a name="snowplow" />
### 5.3.2 js/snowplow.js

Here we come to `Snowplow` module, which can be considered as third entry point,
but you may have as many `Snowplow` objects as many namespace objects you have.
This is obviously because each `snowplow_name_here.q` is `Snowplow` object.

`snowplow.Snowplow` is side-effecting constructor (I cannot see why it should 
be a constructor and not a simple function). Its side-effect is attaching 
callback on `DOMContentLoaded` which executes all functions inside 
`bufferFlushers` and `registeredOnLoadHandlers` (we will talk about these 
arrays later).

But real purpose of `Snowplow` constructor is to call constructor for
`InQueueManager`. `Snowplow` creates callback, creates global `window.Snowplow`
object (not used anywhere except sync tracker), initalizes several variables
like `snowplowMutState` to pass them around and calls `InQueueManager`
constructor (I cannot see why it cannot be plain function as well).

<a name="in_queue" />
### 5.3.3 js/in_queue.js

`InQueueManager` object is exactly that `q` property on namespace object.

Single method of `InQueueManager` is `push` aliased to `applyAsyncFunction`.
It doesn't take any particular arguments, but instead it works with JS special
`arguments` object, which is array-like structure representing everything
passed to function. Usually its `arguments` has just one element and this
element is `arguments` object of very top-level function `p[i]` defined in our
tag, so if we look at [example initialization](example-newtracker) like it is 
array, it should look like following:

```javascript 
// nested one-element array
[["newTracker", "cf", "d3rkrsqld9gmqf.cloudfront.net", { appId: "cfe23a", platform: "mob" })]]
```

Also `InQueueManager` is responsible for holding all named trackers (we'll
discuss tracker objects below). And once `applyAsyncFunction` receives
`newTracker` function - it creates it and puts into internal
`trackerDictionary` object. When it receives another functions (tracking method 
calls) - it just decides on which tracker this method should be called and
actually invokes it.

<a name="tracker" />
### 5.3.4 js/tracker.js

When loop in `push` encounters `newTracker` method - it invokes `Tracker` 
constructor. `Tracker` is the biggest class and core of javascript tracker
logic. But most of `Tracker` methods are self-descriptive, every method in
returned object is public and can be called using
`snowplow_name_here('someMethod', foo, bar)` syntax. Also it has short
side-effecting constructor and lot of private helper methods.


<a name="core" />
### 5.3.5 core/lib/core.ts

Tracker's `core` object is part of separate project published as
[`snowplow-tracker-core`][npm-core]. Its purpose is to serve as high-level
interface over [Snowplow Tracker Protocol][tracker-protocol], independent of 
particular JavaScript implementation or runtime (browser or node). All its
logic concentrated around HTTP payload. Most important thing about `core` is
its callback. Whenever `track` method is called (at the end of every tracking
method)at the end of every tracking method) - this callback is getting called.

Particularly in `Tracker` this callback sends payload data into
`OutQueueManager`.

<a name="out_queue" />
### 5.3.6 js/out_queue.js

`OutQueueManager` is transport layer for tracker. Here's the end of the world
for events. It connects to collector using chosen HTTP method and performs
request with payload created in `core` and `Tracker`.

<a name="tests" />
## 5.4 Tests

### 5.4.1 Automated testing

For testing javascript tracker uses [intern][intern] testing framework.

You can install it with npm as part of devDependencies: 
`cd snowplow-javascript-tracker && npm install`.

Intern provides two strategies for testing: unit and functional tests.

You can run unit tests using `grunt intern:nonfunctional`. You don't need any
other auxiliary tools for running unit tests, they're pure JavaScript and can
be executed in node environment.

Functional tests are bit different. We have two functional test suites:
`functional` and `integration`. Functional tests work by emulating user
interactions with browser, therefore they need some "browser emulator". We're
using [Sauce Labs][saucelabs] for emulating browser - it provides many cool
features such multiple browser/OS environments, interaction recording, etc. You
need to have an account on Sauce Labs (it has no free plans, sadly) and provide 
`SAUCE_ACCESS_KEY` and `SAUCE_USERNAME` environment variables. Providing it you
can run functional suite with `grunt intern:functional`.

Another functional suite called `integration`. It not just emulates user
interactions, but also sends data (tracking payload) to [emulated
collector][snowplow-collector]. This collector emulator is primitive node.js
middleware that write down GET-payloads as JSON to temporary file, which then 
can be parsed by test runner. You can find this middleware at 
`tests/integration/request_recorder.js`. But some more preparation is required.
Problem here is that integration suite executing on remote Sauce Lab server,
whereas requirest recorder runs on local machine. To solve this we're using 
[Ngrok][ngrok]. This tool tunnels your local port to some public acessible 
machine, which means your collector emulator will be available at public host.
You'll also need to set this host as `SUBDOMAIN` environment variable and grunt
will generate html page out of `tests/pages/integration-template.html` with
appropriate domain (`SUBDOMAIN.ngrok.io`) as collector.

### 5.4.2 Manual testing

TODO: mention async-small etc

<a name="build" />
## 5.5 Build

We're using Grunt as our build system.



[grunt]: http://gruntjs.com/
[browserify]: http://browserify.org/
[commonjs]: https://en.wikipedia.org/wiki/CommonJS
[npmjs]: https://www.npmjs.com/
[intern]: https://theintern.github.io/
[typescript]: https://www.typescriptlang.org/
[saucelabs]: https://saucelabs.com/open-source#automated-testing-platform
[ngrok]: https://ngrok.com/

[example-newtracker]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker#22-initialising-a-tracker
[npm-core]: https://www.npmjs.com/package/snowplow-tracker-core
[tracker-protocol]: https://github.com/snowplow/snowplow/wiki/snowplow-tracker-protocol
[snowplow-collector]: https://github.com/snowplow/snowplow/wiki/Setting-up-a-Collector
