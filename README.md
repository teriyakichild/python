<p align="center">
<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->
[![All Contributors](https://img.shields.io/badge/all_contributors-8-orange.svg?style=flat-square)](#contributors-)
<!-- ALL-CONTRIBUTORS-BADGE:END -->
  <a href="https://app.logdna.com">
    <img height="95" width="202" src="https://raw.githubusercontent.com/logdna/artwork/master/logo%2Bpython.png">
  </a>
  <p align="center">Python package for logging to <a href="https://app.logdna.com">LogDNA</a></p>
</p>

---

* [Installation](#installation)
* [Setup](#setup)
* [Usage](#usage)
  * [Usage with File Config](#usage-with-file-config)
* [API](#api)
  * [LogDNAHandler(key: string, [options: dict])](#logdnahandlerkey-string-options-dict)
    * [key](#key)
    * [options](#options)
  * [log(line, [options])](#logline-options)
    * [line](#line)
    * [options](#options-1)
* [Development](#development)
  * [Scripts](#scripts)
* [License](#license)

## Installation

```bash
$ pip install logdna
```

## Setup
```python
import logging
from logdna import LogDNAHandler
import os

# Set your key as an env variable
# then import here, its best not to
# hard code your key!
key=os.environ['INGESTION_KEY']

log = logging.getLogger('logdna')
log.setLevel(logging.INFO)

options = {
  'hostname': 'pytest',
  'ip': '10.0.1.1',
  'mac': 'C0:FF:EE:C0:FF:EE'
}

# Defaults to False; when True meta objects are searchable
options['index_meta'] = True

test = LogDNAHandler(key, options)

log.addHandler(test)

log.warning("Warning message", {'app': 'bloop'})
log.info("Info message")

```
_**Required**_
* [LogDNA Ingestion Key](https://app.logdna.com/manage/profile)

_**Optional**_
* Hostname - ([string][]) - max length 32 chars
* MAC Address - ([string][])
* IP Address - ([string][])
* Max Length - ([bool][]) - formatted as options['max_length']
* Index Meta - ([bool][]) - formatted as options['index_meta']

## Usage

After initial setup, logging is as easy as:
```python
# Simplest use case
log.info('My Sample Log Line')

# Add a custom level
log.info('My Sample Log Line', { 'level': 'MyCustomLevel' })

# Include an App name with this specific log
log.info('My Sample Log Line', { 'level': 'Warn', 'app': 'myAppName'})

# Pass associated objects along as metadata
meta = {
    'foo': 'bar',
    'nested': {
        'nest1': 'nested text'
    }
}

opts = {
    'level': 'warn',
    'meta': meta
}

log.info('My Sample Log Line', opts)
```

### Usage with File Config

To use [LogDNAHandler](#logdnahandlerkey-string-options-dict) with [fileConfig][] (e.g., in a Django `settings.py` file):

```python
import os
import logging
from logdna import LogDNAHandler #  required to register `logging.handlers.LogDNAHandler`

LOGGING = {
    # Other logging settings...
    'handlers': {
        'logdna': {
            'level': logging.DEBUG,
            'class': 'logging.handlers.LogDNAHandler',
            'key': os.environ.get('LOGDNA_INGEST_KEY'),
            'options': {
                'app': '<app name>',
                'env': os.environ.get('ENVIRONMENT'),
                'index_meta': <True|False>,
            },
        },
    },
    'loggers': {
        '': {
            'handlers': ['logdna'],
            'level': logging.DEBUG
        },
    },
}
```

(This example assumes you have set environment variables for `ENVIRONMENT` and `LOGDNA_INGEST_KEY`)

## API

### LogDNAHandler(key: [string][], [options: [dict][]])

#### key

* _**Required**_
* Type: [string][]
* Values: `YourAPIKey`

The [LogDNA API Key](https://app.logdna.com/manage/profile) associated with your account.

#### options

##### app

* _Optional_
* Type: [string][]
* Default: `''`
* Values: `YourCustomApp`
* Max Length: `32`

The default app passed along with every log sent through this instance.

##### env

* _Optional_
* Type: [string][]
* Default: `''`
* Values: `YourCustomEnv`
* Max Length: `32`

The default env passed along with every log sent through this instance.

##### hostname

* _Optional_
* Type: [string][]
* Default: `''`
* Values: `YourCustomHostname`
* Max Length: `32`

The default hostname passed along with every log sent through this instance.

##### include_standard_meta

* _Optional_
* Type: [bool][]
* Default: `False`

Python [LogRecord][] objects includes language-specific information that may be useful metadata in logs.
Setting `include_standard_meta` to `True` automatically populates meta objects with `name`, `pathname`, and `lineno`
from the [LogRecord][].


##### index_meta

* _Optional_
* Type: [bool][]
* Default: `False`

We allow meta objects to be passed with each line. By default these meta objects are stringified and not searchable, and are only displayed for informational purposes.

If this option is set to True then meta objects are parsed and searchable up to three levels deep. Any fields deeper than three levels are stringified and cannot be searched.

*WARNING* If this option is True, your metadata objects MUST have consistent types across all log messages or the metadata object may not be parsed properly.


##### level

* _Optional_
* Type: [string][]
* Default: `Info`
* Values: `Debug`, `Trace`, `Info`, `Warn`, `Error`, `Fatal`, `YourCustomLevel`
* Max Length: `32`

The default level passed along with every log sent through this instance.

##### verbose

* _Optional_
* Type: [string][] or [bool][]
* Default: `True`
* Values: `False` or any level

Sets the verbosity of log statements for failures.

##### request_timeout

* _Optional_
* Type: [int][]
* Default: `30000`

The amount of time (in ms) the request should wait for LogDNA to respond before timing out.

##### tags

* _Optional_
* Type: [list][]&lt;[string][]&gt;
* Default: `[]`

List of tags used to dynamically group hosts.  More information on tags is available at [How Do I Use Host Tags?](https://docs.logdna.com/docs/logdna-agent#section-how-do-i-use-host-tags-)

##### url

* _Optional_
* Type: [string][]
* Default: `'https://logs.logdna.com/logs/ingest'`

A custom ingestion endpoint to stream log lines into.

### log(line, [options])

#### line

* _Required_
* Type: [string][]
* Default: `''`
* Max Length: `32000`

The log line to be sent to LogDNA.

#### options

##### level

* _Optional_
* Type: [string][]
* Default: `Info`
* Values: `Debug`, `Trace`, `Info`, `Warn`, `Error`, `Fatal`, `YourCustomLevel`
* Max Length: `32`

The level passed along with this log line.

##### app

* _Optional_
* Type: [string][]
* Default: `''`
* Values: `YourCustomApp`
* Max Length: `32`

The app passed along with this log line.

##### env

* _Optional_
* Type: [string][]
* Default: `''`
* Values: `YourCustomEnv`
* Max Length: `32`

The environment passed with this log line.

##### meta

* _Optional_
* Type: [dict][]
* Default: `None`

A standard dictonary containing additional metadata about the log line that is passed. Please ensure values are JSON serializable.

**NOTE**: Values that are not JSON serializable will be removed and the respective keys will be added to the `__errors` string.

##### index_meta

* _Optional_
* Type: [bool][]
* Default: `False`

We allow meta objects to be passed with each line. By default these meta objects will be stringified and will not be
searchable, but will be displayed for informational purposes.

If this option is turned to true then meta objects will be parsed and will be searchable up to three levels deep. Any fields deeper than three levels will be stringified and cannot be searched.

*WARNING* When this option is true, your metadata objects across all types of log messages MUST have consistent types or the metadata object may not be parsed properly!

##### timestamp

* _Optional_
* Type: [float][]
* Default: [time.time()][]

The time in seconds since the epoch to use for the log timestamp. It must be within one day or current time - if it is not, it is ignored and time.time() is used in its place.


## Development

This project makes use of the [poetry][] package manager for local development.

### Scripts

**lint**
Run linting rules w/o attempting to fix them

```shell
$ poetry run lint
```


**lint:fix**

Run lint rules against all local python files and attempt to fix where possible.


```shell
$ poetry run lint:fix
```

**test**:

Runs all unit tests and generates coverage reports

```shell
poetry run test
```

## Contributors ✨

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tr>
    <td align="center"><a href="https://github.com/respectus"><img src="https://avatars.githubusercontent.com/u/1046364?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Muaz Siddiqui</b></sub></a><br /><a href="https://github.com/logdna/python/commits?author=respectus" title="Code">💻</a> <a href="https://github.com/logdna/python/commits?author=respectus" title="Documentation">📖</a> <a href="https://github.com/logdna/python/commits?author=respectus" title="Tests">⚠️</a></td>
    <td align="center"><a href="https://github.com/smusali"><img src="https://avatars.githubusercontent.com/u/34287490?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Samir Musali</b></sub></a><br /><a href="https://github.com/logdna/python/commits?author=smusali" title="Code">💻</a> <a href="https://github.com/logdna/python/commits?author=smusali" title="Documentation">📖</a> <a href="https://github.com/logdna/python/commits?author=smusali" title="Tests">⚠️</a></td>
    <td align="center"><a href="https://github.com/vilyapilya"><img src="https://avatars.githubusercontent.com/u/17367511?v=4?s=100" width="100px;" alt=""/><br /><sub><b>vilyapilya</b></sub></a><br /><a href="https://github.com/logdna/python/commits?author=vilyapilya" title="Code">💻</a> <a href="#maintenance-vilyapilya" title="Maintenance">🚧</a> <a href="https://github.com/logdna/python/commits?author=vilyapilya" title="Tests">⚠️</a></td>
    <td align="center"><a href="https://github.com/mikehu"><img src="https://avatars.githubusercontent.com/u/981800?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Mike Hu</b></sub></a><br /><a href="https://github.com/logdna/python/commits?author=mikehu" title="Documentation">📖</a></td>
    <td align="center"><a href="http://codedependant.net/"><img src="https://avatars.githubusercontent.com/u/148561?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Eric Satterwhite</b></sub></a><br /><a href="https://github.com/logdna/python/commits?author=esatterwhite" title="Code">💻</a> <a href="https://github.com/logdna/python/commits?author=esatterwhite" title="Documentation">📖</a> <a href="https://github.com/logdna/python/commits?author=esatterwhite" title="Tests">⚠️</a> <a href="#tool-esatterwhite" title="Tools">🔧</a></td>
    <td align="center"><a href="http://dev.utek.pl/"><img src="https://avatars.githubusercontent.com/u/128036?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Łukasz Bołdys (Lukasz Boldys)</b></sub></a><br /><a href="https://github.com/logdna/python/commits?author=utek" title="Code">💻</a> <a href="https://github.com/logdna/python/issues?q=author%3Autek" title="Bug reports">🐛</a></td>
    <td align="center"><a href="https://github.com/baronomasia"><img src="https://avatars.githubusercontent.com/u/4133158?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Ryan</b></sub></a><br /><a href="https://github.com/logdna/python/commits?author=baronomasia" title="Documentation">📖</a></td>
  </tr>
  <tr>
    <td align="center"><a href="https://github.com/LYHuang"><img src="https://avatars.githubusercontent.com/u/14082239?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Mike Huang</b></sub></a><br /><a href="https://github.com/logdna/python/commits?author=LYHuang" title="Code">💻</a> <a href="https://github.com/logdna/python/issues?q=author%3ALYHuang" title="Bug reports">🐛</a></td>
  </tr>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!

## License

MIT © [LogDNA](https://logdna.com/)
Copyright © 2017 [LogDNA][], released under an MIT license. See the [LICENSE](./LICENSE) file and [https://opensource.org/licenses/MIT](https://opensource.org/licenses/MIT)


*Happy Logging!*

[bool]: https://docs.python.org/3/library/stdtypes.html#boolean-values
[dict]: https://docs.python.org/3/library/stdtypes.html#mapping-types-dict
[int]: https://docs.python.org/3/library/functions.html#int
[float]: https://docs.python.org/3/library/functions.html#float
[string]: https://docs.python.org/3/library/string.html
[list]: https://docs.python.org/3/library/stdtypes.html#list
[time.time()]: https://docs.python.org/3/library/time.html?#time.time
[poetry]: https://python-poetry.org
[LogDNA]: https://logdna.com/
[LogRecord]: https://docs.python.org/2/library/logging.html#logrecord-objects
[fileConfig]: https://docs.python.org/2/library/logging.config.html#logging.config.fileConfig
