# Voicemail FSM Documentation

## lib/fsm.js

[Voicemail FSM](https://github.com/asterisk/node-voicemail-fsm) is a finite state machine responsible for enforcing all business logic for the [Voicemail](voicemail.md) application.

It accepts a [StasisStart](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisStart) event and an object representing the channel that entered the voicemail application and takes control of the application. All cleanup is done by this module or modules used by it. Once a new instance of a finite state machine has been created, voicemail no longer has to interact with the finite state machine module.

After an instance of the finite state machine is created, the incoming channel is answered and event handlers for a [StasisEnd](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisEnd) and [ChannelDtmfReceived](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-ChannelDtmfReceived) events are registered. Upon the channel hanging up, the finite state machine will clean itself up using the [StasisEnd](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisEnd) event handler. Every time a user interacts with the application using a DTMF key, the [ChannelDtmfReceived](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-ChannelDtmfReceived) event handler will handle converting the input to an action the finite state machine can respond to. A mapping of DTMF keys to valid actions is defined in the config.json file found at the root of the [Voicemail](voicemail.md) application.

The finite state machine then uses the [Auth](auth.md) module to load the mailbox the user is leaving a message for using the context and mailbox number passed to the [Stasis](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+Application_Stasis) application via application arguments. Whether the mailbox owner is busy or unavailable is attached to the mailbox at this point. See [Voicemail](voicemail.md) for an example dialplan that passes a context, mailbox number, and busy status to the voicemail [Stasis](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+Application_Stasis) application.

The finite state machine then uses the [Mailbox](mailbox.md) module to prepare to record the channel. The mailbox module then plays a help prompt to inform the user of whether the mailbox owner is busy or unavailable as well as instructions on leaving a message. A user may press a DTMF key defined in the config.json found in the root of the [Voicemail](voicemail.md) application to stop the help prompt played by the mailbox module and go straight to recording a message. After the help prompt finishes, whether by completing or due to the user skipping it, a recording is initiated on the channel. The user can then either hang up or use the same DTMF key defined in the config.json file to end the recording. If a user chooses to end the recording using the defined DTMF key, a prompt is played on the channel using the [Prompt](prompt.md) module to inform the user that the application is exiting and that the channel will be hung up momentarily.

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

Voicemail FSM depends on the following voicemail modules:

- [Auth](auth.md)
- [Mailbox](mailbox.md)
- [Config](config.md)
- [Prompt](prompt.md)
- [Logging](logging.md)
