<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**Ruby tracker**](Ruby-tracker-setup)

## Contents

- 1. [Overview](#overview)  
- 2 [Dependencies](#dependencies)
- 3. [Setup](#setup)
  - 3.1 [RubyGems](#rubygems)
  - 3.2 [Installation](#installation)

<a name="overview" />

## 1. Overview

The [Snowplow Ruby Tracker](https://github.com/snowplow/snowplow-ruby-tracker) lets you add analytics to your [Ruby][ruby] applications and gems.

Setting up the tracker should be relatively straightforward if you are familiar with Ruby development.

Ready? Let's get started.

[Back to top](#top)

<a name="dependencies" />

## 2. Dependencies

The Snowplow Ruby Tracker is compatible with Ruby versions 1.9.3, 2.0.0, and 2.1.0.

To make the Snowplow Ruby Tracker work with as many different Ruby programs as possible, we have tried to keep external dependencies to a minimum. There are only two external dependencies currently:

* [contracts][contracts] - Ruby package that allows one to declare constraints on function parameters and return values.
* [webmock][webmock] - HTTP library used to stub requests for testing.

These dependencies can be installed from the package manager of the host system or through RubyGems.

[Back to top](#top)

<a name="setup" />

## 3. Setup

<a name="rubygems" />

### 3.1 RubyGems

The Snowplow Ruby Tracker is published to [RubyGems][rubygems], the Ruby community's gem hosting service. 
This makes it easy to either install the tracker locally, or to add it as a dependency into your own Ruby app.

<a name="installation" />

### 3.2 Installation

To install the Snowplow Ruby Tracker locally:

    $ gem install snowplow-tracker

To add the Snowplow Tracker as a dependency to your own Ruby gem, edit your gemfile and add:

```ruby
gem 'snowplow-tracker'
```

Done? Now read the [Ruby Tracker API](Ruby-Tracker) to start tracking events.

[Back to top](#top)

[ruby]: https://www.ruby-lang.org/en/
[contracts]: http://rubygems.org/gems/contracts
[webmock]: https://rubygems.org/gems/webmock
[rubygems]: http://rubygems.org/
