
# Voicemail Auth Documentation

## lib/auth.js

[Voicemail Auth](https://github.com/asterisk/node-voicemail-auth) is a module responsible for enforcing mailbox authentication for the [Voicemail](voicemail.md) and [Voicemail Main](voicemail-main.md) applications.

When called with all required dependencies, it exposes a create method that allows the creation of an authenticator object. The create method accepts an object representing the channel that entered the [Voicemail](voicemail.md) or [Voicemail Main](voicemail-main.md) application as well as a boolean value to determine if authentication should be skipped. If authentication should be skipped, the mailbox will still be loaded using a domain and mailbox number, but no password may be verified for it. This is useful for the [Voicemail](voicemail.md) application.

The authenticator object returned by the create method exposes an API that returns promises but uses a finite state machine internally to control interactions with other voicemail modules. Once an authenticator has been created, the finite state machine is initialized and an event handler is registered for a [StasisEnd](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisEnd) event to handle all required cleanup upon the channel hanging up. The authenticator object exposes the following methods:

### init

When init is called, a promise is returned that will be resolved once the internal final state machine raises an AuthenticatorLoaded event. init does this by registering an event handler for this event, and calling handle on the finite state machine for the 'init' action, giving it the domain and mailbox number to use for initializing the authenticator. Any errors in the finite state machine will instead raise an Error event, and the promise will be rejected with the error.

Inside the internal finite state machine, the domain will first be loaded from the database followed by the mailbox using the [Data](data.md) module. If the domain is not found, a ContextNotFound error will be raised so the promise can be rejected and the finite state machine will transition to a 'done' state where no further input can be accepted. If the mailbox number is not found, a MailboxNotFound error will be raised so that the promise can be rejected and the finite state machine will transition to an 'unknownMailbox' state so that the setMailbox method can be called on the authenticator once the user has provided another mailbox number.

If both the context and mailbox are successfully loaded, the finite state machine will transition to an 'authenticating' state so that the authenticate method can be called on the authenticator and an AuthenticatorLoaded event will be raised so that the promise can be resolved.

Whether the promise is resolved or rejected, all event handlers registered against the finite state machine will be unregistered.

### setMailbox

This method should be called after the promise returned by init has been rejected with a MailboxNotFound error.

When setMailbox is called, a promise is returned that will be resolved once the internal final state machine raises an AuthenticatorLoaded event. setMailbox does this by registering an event handler for this event, and calling handle on the finite state machine for the 'setMailbox' action, giving it the mailbox number to use to load the mailbox. Any errors in the finite state machine will instead raise an Error event, and the promise will be rejected with the error.

Inside the finite state machine, the mailbox will first be loaded from the database using the [Data](data.md) module. If the mailbox number is not found, a MailboxNotFound error is raised and the finite state machine stays in its current state. Otherwise, the finite state machine transitions to an 'authenticating' state so that the authenticate method can be called on the authenticator and an AuthenticatorLoaded event will be raised so that the promise can be resolved.

Whether the promise is resolved or rejected, all event handlers registered against the finite state machine will be unregistered.

### authenticate

This method should be called after the promise returned by either init or setMailbox has been resolved.

Inside the finite state machine, once the state is transitioned to the 'authenticating' state, the boolean value passed to the authenticator to determine if authentication should be skipped is checked. If authentication should be skipped, the finite state machine transitions to a 'done' state where no more input can be processed. Otherwise, a prompt is played using the [Prompt](prompt.md) module to inform the user that a password should be entered.

When authenticate is called, a promise is returned that will be resolved once the internal final state machine raises an Authenticated event. authenticate does this by registering an event handler for this event, and calling handle on the finite state machine for the 'authenticate' action, giving it the password to use to authenticate the mailbox. Any errors in the finite state machine will instead raise an Error event, and the promise will be rejected with the error.

Inside the finite state machine, the authenticate action is called with a password which is verified against the mailbox password. If the password is correct, an Authenticated event is raised and the finite state machine transitions to a 'done' state where no more inputs can be processed. Otherwise, a prompt is played using the [Prompt](prompt.md) module to inform the user that the password was incorrect and an InvalidPassword error is raised.

Whether the promise is resolved or rejected, all event handlers registered against the finite state machine will be unregistered.

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

Voicemail Auth depends on the following voicemail modules:

- [Data](data.md)
- [Prompt](prompt.md)
