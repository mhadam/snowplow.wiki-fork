<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Snowplow Analytics SDK**](Snowplow-Analytics-SDK) » Python Analytics SDK setup

## Contents

1. [Overview](#overview)  
2. [Compatibility](#compatibility)  
3. [Setup](#setup)  
  3.1 [PyPi](#pypi)  
  3.2 [pip](#pip)  
  3.3 [easy_install](#easy_install)  


<a name="overview" />

## 1. Overview

The [Snowplow Analytics SDK for Python](https://github.com/snowplow/snowplow-python-analytics-sdk) lets you work with [Snowplow enriched events](https://github.com/snowplow/snowplow/wiki/canonical-event-model) in your Python event processing, data modeling and machine-learning jobs. 
You can use this SDK with [Apache Spark][spark], [AWS Lambda](https://aws.amazon.com/lambda/), and other Python-compatible data processing frameworks.

The SDK should be relatively straightforward to setup if you are familiar with Python development.

<a name="compatibility" />

## 2. Compatibility

Snowplow Python Analytics SDK was tested with Python of versions: 2.7, 3.3, 3.4, 3.5.
As analytics SDKs supposed to be used heavily in conjunction with data-processing engines such as [Apache Spark][spark], our goal is to maintain compatibility with all versions that PySpark supports.
Whenever possible we try to maintain compatibility with broader range of Python versions and computing environments.
This is achieved mostly by minimazing and isolating third-party dependencies and libraries.

There are only one external dependency currently:

* [Boto3][boto3] - AWS Python SDK that used to provide access to Event Load Manifests.

These dependencies can be installed from the package manager of the host system or through PyPi.

[Back to top](#top)

<a name="setup" />

## 3. Setup

<a name="pypi" />

### 3.1 PyPI

The Snowplow Python Analytics SDK is published to [PyPI][pypi], the the official third-party software repository for the Python programming language.

This makes it easy to either install the SDK locally, or to add it as a dependency into your own Python app or Spark job.

<a name="pip" />

### 3.2 pip

To install the Snowplow Python Analytics SDK locally, assuming you already have Pip installed:

```bash
$ pip install snowplow_python_sdk --upgrade
```

To add the Snowplow Analytics SDK as a dependency to your own Python app, edit your `requirements.txt` and add:

```python
snowplow_analytics_sdk==0.2.0
```

<a name="easy_install" />

### 3.3 easy_install

If you are still using easy_install:

```bash
$ easy_install -U snowplow_python_sdk
```

Done? Now read the [Python Analytics SDK API](Python-Analytics-SDK) to start analyzing events data.

[python]: http://www.python.org/
[boto3]: https://aws.amazon.com/sdk-for-python/
[pypi]: https://pypi.python.org/
[spark]: http://spark.apache.org/
