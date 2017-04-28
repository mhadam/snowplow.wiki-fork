[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**JavaScript Tracker**](Javascript-tracker-setup) > [Modifying Snowplow.js](Modifying-Snowplow-js)

## Overview

It may be necessary for you to make changes to `snowplow.js`. If you do, then you will also need to minify your updated `snowplow.js` prior to deploying it. This guide covers how to do this.

To continue with this guide, you will need the following

* Access to a Unix-like command line
* Familiarity with and access to [Git][git]
* Node.js version 10 or above installed
* npm version 1.4.0 or above installed
* Some level of technical ability _ta2_, where `ta1 < ta2 < ninja`

See [this page][node-and-npm-installation] for guidance help installing node and npm.

Once you have those ready, please read on...

## 1. Check out the source code

First please download the source code to your development machine:

    $ cd ~/development
    $ git clone git@github.com:snowplow/snowplow-javascript-tracker.git
	$ cd snowplow-javascript-tracker
    $ npm install
    $ grunt
    $ cd dist
    $ ls
    bundle.js snowplow.js sp.js

In the listing above, `bundle.js` is the [Browserified][browserify] JavaScript;  `snowplow.js` is identical to `bundle.js` but has a banner comment at the top; and `sp.js` is the minified version.

## 2. Update `snowplow.js`

Making changes to `snowplow.js` (and testing them) is out of scope of this document. A brief overview of the steps it involves:

* Make whatever changes you want to inside `snowplow-javascript-tracker/src/js`
* Run `$ grunt` from the `snowplow-javascript-tracker` directory to rebuild the files in `snowplow-javascript-tracker/dist`
* Use the example page `snowplow-javascript-tracker/examples/web/async/async-small.html` to test your changes locally. You will probably want to edit that file, replacing `"//d1fc8wv8zag5ca.cloudfront.net/2.0.0/sp.js"` with `"../../dist/snowplow.js"`, so that it loads your modified copy of `snowplow.js`.

## 3. Upload the minified JavaScript

Use your standard asset pipelining strategy to upload the minified `sp.js` JavaScript to your servers. Note that to avoid "mixed content" warnings, Snowplow expects the `sp.js` JavaScript to be available both via HTTP and via HTTPS.

## 4. Test your updated JavaScript

As a final step, you will want to just check that your customised `sp.js` JavaScript is working okay.

To do this, please follow the testing instructions on [Testing the Javascript tracker is firing](Testing%20the%20Javascript%20tracker%20is%20firing).

## Next steps

Finished setting up the [JavaScript Tracker](javascript-tracker-setup)? Then you are ready to [setup EmrEtlRunner](Setting-up-Snowplow#wiki-step3).

Return to the [setup guide](Setting-up-Snowplow).

[node-and-npm-installation]: https://gist.github.com/isaacs/579814
[browerify]: http://browserify.org/
