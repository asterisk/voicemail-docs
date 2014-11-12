# Voicemail Prompt Documentation

## lib/prompt.js

[Voicemail Prompt](https://github.com/asterisk/node-voicemail-prompt) is a module responsible for playing sequences of sounds and allowing the sequence to be stopped at any point.

When called with all required dependencies, it exposes a create method that allows the creation of a prompt object. The create method accepts an array of sounds, an object representing the channel that entered the [Voicemail](voicemail.md) or [Voicemail Main](voicemail-main.md) application as well as an object to be used to substitute values in the sounds configured in the config.json file found at the root of either the [Voicemail](voicemail.md) or [Voicemail Main](voicemail-main.md) application.

The prompt object returned by the create method exposes an API that returns promises but uses a finite state machine internally to control interactions with ARI. Once a prompt has been created, the finite state machine is initialized, an event handler is registered for a [StasisEnd](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisEnd) event to handle all required cleanup upon the channel hanging up, and a connection to ARI is established using the [Config](config.md) and [ARI Wrapper](ari.md) modules. Once the connection to ARI has been established and a client loaded, the state machine transitions to a 'processing' state where it awaits further input. The prompt object exposes the following methods:

### play

When play is called, a promise is returned that will be resolved with the boolean value true if the finite state machine raises a PromptFinished event, or false if the finite state machine raises a PromptStopped event. play does this by registering event handlers for these events, and calling handle on the finite state machine for the 'play' action. Any errors in the finite state machine will instead raise an Error event, and the promise will be rejected with the error.

Inside the internal finite state machine, the array of sounds is cloned so that it can be modified without affecting the original, all substitutions are performed using the object provided to the create method, and finally the state machine transitions to a 'playing' state and calls handle for the 'next' action so that the next sound in the sequence can begin being played on the channel.

When next is called in the 'playing' state, the first sound in the current sequence is removed from the array for playback. If all sounds have been played or if the current sound allows skipping and the sequence has been stopped, handle is called on the finite state machine for the 'finished' action. Otherwise, the current sound is played on the channel using ARI and a [PlaybackFinished](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-PlaybackFinished) event handler is registered against the playback. Once the PlaybackFinished event fires, handle is called on the state machine for the 'finished' action.

When the finished action is called, handle is called on the finite state machine for the 'next' action after a timeout configured through the current sound's postSilence property if more sounds exist in the sequence. Otherwise if all sounds have been played to completion, a PromptFinished event is raised. If the sequence was stopped at some point, a PromptStopped event is raised instead. Regardless of which event is raised, the state machine transitions to a 'done' state where no more inputs can be processed.

Whether the promise is resolved or rejected, all event handlers registered against the finite state machine will be unregistered.

### stop

This method can be called at any point after play has been called.

When stop is called, a property is set to mark the prompt sequence as stopped so that any further calls to the 'next' action will know to skip the sound. If the current sound allows skipping, it is stopped using ARI.

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

Voicemail Prompt depends on the following voicemail modules:

- [Config](config.md)
- [ARI Wrapper](ari.md)
