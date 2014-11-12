# Voicemail Config Documentation

## lib/config.js

[Voicemail Config](https://github.com/asterisk/node-voicemail-config) is a module responsible for application configuration for [Voicemail](voicemail.md) and [Voicemail Main](voicemail-main.md) as well as mailbox configuration.

When called with an instance of the [Data](data.md) module, it returns an object with methods that can be used to get either application configuration or mailbox configuration. The object exposes the following methods:

### getAppConfig

When getAppConfig is called, the configuration loaded from the config.json file at the root of either the [Voicemail](voicemail.md) or [Voicemail Main](voicemail-main.md) application is returned.

### getMailboxConfig

When getMailboxConfig is called with a mailbox, configuration for that mailbox is returned. Configuration items found in the application configuration are first loaded. Configuration items for the context associated with the provided mailbox are then loaded from the database using the [Data](data.md) module and applied, potentially overriding configuration items from the application configuration. Finally, configuration items for the mailbox are loaded from the database using the [Data](data.md) module and applied, potentially overriding configuration items from the context configuration.

This scheme allows generic configuration at the application level that can be either overridden or added to at the context or mailbox level.

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

Voicemail Config depends on the following voicemail modules:

- [Data](data.md)
