<a name="top" />

## Contents

- 1. [Installation](#install)
  - 1.1 [Composer](#composer)

<a name="install" />

## 1. Installation

Currently the only supported method of installation is through Composer.  For a tutorial on setting up a PHP project with Composer please follow this [link](https://getcomposer.org/doc/00-intro.md). 

<a name="composer" />

### 1.1 Composer

Using Composer to manage your dependencies, simply add the Snowplow PHP Tracker to your project by including it in your composer.json file as a dependency.

```json
{
    "require": {
        "snowplow/snowplow-tracker": "0.2.1"
    }
}
```

Assuming you have Composer setup correctly in the root of your project.
Type the following command line argument:

```sh
composer install #If composer has not been run yet
composer update #If composer dependencies are already installed
```

This will install the Snowplow Tracker and allow you to initialize a Tracker object:

```PHP
// Bare minimum Tracker initialization.
 
use Snowplow\Tracker\Tracker;
use Snowplow\Tracker\Subject;
use Snowplow\Tracker\Emitters\SyncEmitter;

$emitter = new SyncEmitter("collector_uri");
$subject = new Subject();
$tracker = new Tracker($emitter, $subject);
```