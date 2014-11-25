# Voicemail Main FSM Documentation

## lib/fsm.js

[Voicemail Main FSM](https://github.com/asterisk/node-voicemail-main-fsm) is a finite state machine responsible for enforcing all business logic for the [Voicemail Main](voicemail-main.md) application.

It accepts a [StasisStart](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisStart) event and an object representing the channel that entered the voicemail main application and takes control of the application. All cleanup is done by this module or modules used by it. Once a new instance of a finite state machine has been created, voicemail main no longer has to interact with the finite state machine module.

After an instance of the finite state machine is created, the incoming channel is answered and event handlers for a [StasisEnd](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisEnd) and [ChannelDtmfReceived](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-ChannelDtmfReceived) events are registered. Upon the channel hanging up, the finite state machine will clean itself up using the [StasisEnd](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisEnd) event handler. Every time a user interacts with the application using a DTMF key, the [ChannelDtmfReceived](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-ChannelDtmfReceived) event handler will handle converting the input to an action the finite state machine can respond to. A mapping of DTMF keys to valid actions is defined in the config.json file found at the root of the [Voicemail Main](voicemail-main.md) application.

The finite state machine then uses the [Auth](auth.md) module to load the mailbox the user is an owner of using the context and mailbox number passed to the [Stasis](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+Application_Stasis) application via application arguments. See [Voicemail Main](voicemail-main.md) for an example dialplan that passes a context and a mailbox number to the voicemail-main [Stasis](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+Application_Stasis) application. The [Auth](auth.md) module then prompts the user for a password. Once a password with a length matching the regular expression defined in the config.json file at the root of the [Voicemail Main](voicemail-main.md) application is entered by the user, the finite state machine passes it over to the [Auth](auth.md) module. If the password is invalid, the prompt is repeated and the user may enter the password again. If the password is valid, the state machine creates a [Mailbox Reader](mailbox.md) and enters a state where it can wait for input from the user to navigate through the messages in the mailbox. The [mailbox reader](mailbox.md) concurrently informs the user of how many messages exist in the current folder as well as actions the user may take.

If a user enters DTMF keys that match valid input defined in the config.json file at the root of the [Voicemail Main](voicemail-main.md) application, the finite state machine uses the [mailbox reader](mailbox.md) to perform the operation on the mailbox. The following operations are supported:

- first (play the first message in the current folder)
- replay (replay the last message played)
- next (play the next message in the current folder)
- prev (play the previous message in the current folder)
- delete (delete the last message played)
- changeFolder (change to another folder)

For the majority of these operations, the finite state machine stays in the same state, awaiting further input. If a user chooses to enter the DTMF key mapped to the operation for changing folders however, the finite state machine enters into a state where it awaits a DTMF key mapped to a folder. Again, the DTMF keys must match a regular expression in the config.json file found at the root of the [Voicemail Main](voicemail-main.md) application. Once a valid input is entered by the user, the [mailbox reader](mailbox.md) loads the messages for the folder and informs the user of how many messages exist as well as actions the user may take. The finite state machine then returns to a state where it can wait for input from the user to navigate the messages in the current folder.

### DTMF Keys

Unlike [Voicemail FSM](voicemail-fsm.md), the state machine for Voicemail Main FSM can buffer DTMF keys in order to create multi-key input. For example, the mailbox password is made up of multiple DTMF keys that trigger a single operation (login attempt). The mapping between DTMF key(s) and a valid input can be found in the config.json file at the root of the [Voicemail Main](voicemail-main.md) application.

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

Voicemail Main FSM depends on the following voicemail main modules:

- [Auth](auth.md)
- [Mailbox](mailbox.md)
- [Config](config.md)
- [Logging](logging.md)
