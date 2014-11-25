# Voicemail Logging Documentation

## lib/logging.js

[Voicemail Logging](https://github.com/asterisk/node-voicemail-logging) is a module responsible for creating loggers for use with the [Voicemail](voicemail.md) and [Voicemail Main](voicemail-main.md) applications.

The loggers can be configured using a configuration object to determine to which file the normal and error level logs should be written to. If the directories for the configured files do not exist, they will be created before the logger is initialized.

stdin and stderr are already used as defaults by the loggers when configuring log streams. A set of serializers is also configured so that logging instances fetched from the [Data](data.md) module can be done easily.

### create

create accepts a configuration object found at the root of either the [Voicemail](voicemail.md) or [Voicemail Main](voicemail-main.md) application as well as a root component name and returns a logger configured for voicemail.

The returned logger can be used to create a child logger for a child dependency of the voicemail application using the child method. The child loggers will inherits all the configurations defined on their parent logger.

The logger object will also expose the following methods for logging various levels:

- trace
- debug
- info
- warn
- error
- fatal

All of the above accept a string representing a log message but can also accept an object as well that will be converted to a JSON string when writing to a log stream:

```JavaScript
logger.info({key: value}, 'message');
```

For more information on using the loggers created by this module or to learn how to use the existing log parser, see the [bunyan](https://github.com/trentm/node-bunyan) documentation.

## test/tests.js

All mocha unit tests live here. Tests can be run using the following command:

```bash
$ grunt mochaTest
```

## Gruntfile.js

Grunt tasks for running a linter, unit tests, and code coverage live here. The linter and unit tests can be run using the following command:

```bash
$ grunt
```

The linter can be run using the following command:

```bash
$ grunt jshint
```

Code coverage can be run using the following command:

```bash
$ grunt coverage
```

## Dependencies

Voicemail Logging does not depend on any other voicemail modules.
