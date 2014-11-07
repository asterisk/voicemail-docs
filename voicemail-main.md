# Voicemail Main Application Documentation

## lib/voicemail-main.js

[Voicemail Main](https://github.com/asterisk/node-voicemail-main) is the application responsible for allowing users to listen to messages in their mailbox. Its main responsibility is to bootstrap the application by loading the appropriate dependencies.

After all dependencies have been loaded, it connects to Asterisk via [ARI](https://wiki.asterisk.org/wiki/pages/viewpage.action?pageId=29395573) and establishes a WebSocket connection so that the voicemail-main [Stasis](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+Application_Stasis) application is ready for incoming requests.

It then registers an event handler for a [StasisStart](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisStart) event. Once a channel enters into the application via Stasis, a [finite state machine](https://github.com/asterisk/node-voicemail-main-fsm) to drive voicemail main business logic is spun up and the [StasisStart](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisStart) event as well as an object representing the incoming channel are passed over to it at which point the finite state machines takes control of the application for the given channel.

## app.js

The application can be started by running the following:

```bash
$ node app.js
```

or by loading it as a module and starting it manually:

```JavaScript
var voicemailMain = require('voicemail-main');

voicemailMain.create();
```

This allows the application to be run from a terminal or loaded as part of a larger application.

## extensions.conf

The following dialplan is required for voicemail main to work properly:

```
exten => 9999,1,NoOp()
         same => n,Stasis(voicemail-main,email.com,1000)
         same => n,Hangup()
```

Replace 9999 with your extension, email.com with the mailbox domain, and 1000 with the mailbox extension. The application name can be configured using the config.json file at the root of the voicemail main repository.

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

Voicemail Main depends on the following voicemail modules:

- [Voicemail Main FSM](voicemail-main-fsm.md)
- [Auth](auth.md)
- [Mailbox](mailbox.md)
- [Config](config.md)
- [Prompt](prompt.md)
- [Data](data.md)
- [Notify](notify.md)
- [ARI Wrapper](ari.md)
