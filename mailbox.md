# Voicemail Mailbox Documentation

[Voicemail Mailbox](https://github.com/asterisk/node-voicemail-mailbox) is a module responsible for allowing a user to interact with a mailbox and the messages contained within it. This includes leaving messages as well as listening to messages in various folders for a given mailbox.

## lib/helpers/

### messages.js

This file exposes a module with a create method that allows an object representing a sequence of messages to be created given a mailbox, list of folders, current folder, and required dependencies. This object can be used to interact with messages in a mailbox.

The returned object exposes the following properties:

- latest - a [moment](https://github.com/moment/moment) date object representing the latest message in the current sequence

and the following methods:

#### previousExists

Returns true if at least one previous message exists for the current folder, false otherwise.

#### currentExists

Returns true if a current message exists for the current folder, false otherwise.

#### isEmpty

Returns true if no messages exist for the current folder, false otherwise.

#### isNotEmpty

Returns true if messages exist for the current folder, false otherwise.

#### manyExist

Returns true if more than one message exists for the current folder, false otherwise.

#### getCount

Returns the current count of messages for the current folder.

#### getOrder

Returns the order of the current message in the sequence starting with 1.

#### getCurrentFolder

Returns the current folder for which messages are currently loaded.

#### first

Returns the first message in the sequence for the current folder.

#### next

Returns the next message in the sequence for the current folder.

#### current

Returns the current message in the sequence for the current folder.

#### prev

Returns the previous message in the sequence for the current folder.

#### add

Allows a single or a sequence of messages to be added to the existing sequence of messages. This method ensures that no duplicates will be added and updates the property latest when appropriate.

#### markAsRead

Marks the message as read in the database using the [Data](data.md) module (both the message itself as well as updating the read/unread counts for the mailbox), uses the [Notify](notify.md) module to update MWI counts in Asterisk via ARI and finally moves the message to the Old folder in the database using the [Data](data.md) module. The message remains in the current sequence of messages until the current folder is changed, at which point it will only be loaded for the Old folder.

#### remove

Removes the message from the current sequence of messages.

#### delete

Calls remove to remove the message from the current sequence of messages, deletes the message from the database using the [Data](data.md) module, uses the [Notify](notify.md) module to update MWI counts in Asterisk via ARI and deletes the message in Asterisk via ARI using the [ARI Wrapper](ari.md) module.

#### calculateMenu

Returns a context appropriate sequence of menu options for the current state of the messages sequence that can be used to play menu options that are configured in the config.json file found at the root of either the [Voicemail](voicemail.md) or [Voicemail Main](voicemail-main.md) application. 

#### load

Loads all messages for the current folder using the [Data](data.md) module and loads the mailbox configuration using the [Config](config.md) module if it has not been loaded already.

#### changeFolder

Changes the current folder and uses the load method to load messages for the current folder.

## lib/mailbox.js

This file exposes a module that exposes methods for creating either a mailbox reader or writer. The following methods are exposed:

### createReader

Returns a mailbox reader for the given mailbox and an object representing the channel that entered the [Stasis](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+Application_Stasis) application.

### createWriter

Returns a mailbox writer for the given mailbox and an object representing the channel that entered the [Stasis](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+Application_Stasis) application.

## lib/reader.js

This file exposes a module responsible for listening to messages. The module exposes a create method that accepts a mailbox, channel, and required dependencies and returns an object that can be used to interact with mailbox messages. Internally, a finite state machine is created and initialized. This causes the finite state machine to load all folders, create a message sequence using lib/helpers/messages.js, load all messages for the current folder, and register an event handler for a [StasisEnd](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisEnd) event to handle all required cleanup upon the channel hanging up. The state machine then transitions to an 'intro' state where an introduction message can be played on the channel for the current folder.

Once the 'intro' state is initialized, the finite state machine uses the [Config](config.md) module to load configured sounds and uses the [Prompt](prompt.md) module to play the sounds on the channel. Once the intro sounds have been played, the state machine transitions to a 'ready' state where inputs can be processed.

The returned object exposes the following methods for interacting with the finite state machine:

### first

Plays the first message in the mailbox for the current folder on the given channel. Internally, the finite state machine changes to a 'fetching' state while it loads the message from the sequence of messages using lib/helpers/messages.js and calls handle for the 'playMessage' action once the message has been loaded. This uses the [Prompt](prompt.md) module to play the message along with some information about the message relative to other messages in the mailbox. These information sounds are loaded using the [Config](config.md) module.

The state machine then transitions to a 'processing' state as the message is played. If the message is stopped at any point (due to a user selecting another option for example), the message is stopped and the state machine returns to the 'ready' state to process the new input. If the message completes playing, it is marked as read using lib/helpers/messages.js and handle is called for the 'menu' action. This will play a context appropriate menu to list the options available to the user. If the user chooses to enter an input option, the menu is stopped and the state machine transitions to a 'ready' state to process the input. If the menu completes, the finite state machine simply transitions to the 'ready' state and waits for further input.

### replay

Plays the current message in the mailbox for the current folder on the given channel. Internally, the finite state machine goes through the steps listed above for the first method.

### next

Plays the next message in the mailbox for the current folder on the given channel. Internally, the finite state machine goes through the steps listed above for the first method.

### prev

Plays the previous message in the mailbox for the current folder on the given channel. Internally, the finite state machine goes through the steps listed above for the first method.

### delete

The current message is deleted using lib/helpers/messages.js and the finite state machine transitions to a 'processing' state while sounds are played on the channel using the [Config](config.md) and [Prompt](prompt.md) modules to inform the user that the message was deleted. 

If the sounds are stopped at any point (due to a user selecting another option for example), the sounds are stopped and the state machine returns to the 'ready' state to process the new input. If the sounds complete playing handle is called for the 'menu' action. This will play a context appropriate menu to list the options available to the user. If the user chooses to enter an input option, the menu is stopped and the state machine transitions to a 'ready' state to process the input. If the menu completes, the finite state machine simply transitions to the 'ready' state and waits for further input.

### changeFolder

The finite state machines plays sounds on the channel using the [Config](config.md) and [Prompt](prompt.md) modules to inform the user of the folders available and transitions to a 'changingFolder' state as it waits for further input. 

### submitFolder

After changeFolder has been called, submitFolder can be called to select which folder to change to. This will pass the user option for which folder to change to to the finite state machine and will return a promise that will be resolved once a FolderChanged event is raised by the state machine. If any errors occur, the promise will be rejected with the error instead.

Internally, if the folder option given does not map to a folder the finite state machine will play sounds using the [Config](config.md) and [Prompt](prompt.md) modules to inform the user that their option was invalid. If the option is valid, the state machine will transitions to a 'loadingFolder' state and the folder will be changed and messages loaded using lib/helpers/messages.js. Once the messages are loaded, a FolderChanged event will be raised and the state machine will transitions to the 'intro' state to play introduction sounds for the new folder.

## lib/writer.js

This file exposes a module responsible for leaving messages. The module exposes a create method that accepts a mailbox, channel, and required dependencies and returns an object that can be used to leave a message for the mailbox. Internally, a finite state machine is created and initialized. This causes the finite state machine to load a client to ARI using the [ARI Wrapper](ari.md) module, load the mailbox configuration using the [Data](data.md) module, and register an event handler for a [StasisEnd](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-StasisEnd) event to handle all required cleanup upon the channel hanging up. The state machine then transitions to a 'ready' state and waits for further input.

The returned object exposes the following methods for interacting with the finite state machine:

### record

When record is called, a promise is returned that will be resolved when a RecordingFinished event is raised by the finite state machine. Any error that occurs will instead cause the promise to be rejected with that error.

The finite state machine then plays sounds using the [Config](config.md) and [Prompt](prompt.md) modules to inform the user how to leave a message for the mailbox. The finite state machine transitions to a 'recording' state when the sounds complete either by finishing normally or by being stopped at some point in the sequence.

When the 'recording' state is initialized, a recording is started on the channel via ARI using the [ARI Wrapper](ari.md) module. An event handler is registered for a [RecordingFinished](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-RecordingFinished) event against the recording object. The recording can only be stopped by the user choosing the input option mapped to the stop method below.

Once the [RecordingFinished](https://wiki.asterisk.org/wiki/display/AST/Asterisk+13+REST+Data+Models#Asterisk13RESTDataModels-RecordingFinished) event handler is called, the finite state machine transitions to a 'recordingFinished' state and a RecordingFinished event is raised by the finite state machine to allow the promise returned by record to be resolved.

### stop

If called while the finite state machine is playing instructions to the user, the sound sequence associated with the instructions is stopped, and the recording is started. If called during a recording, the finite state machine transitions to a 'stoppingRecording' state. When the 'stoppingRecording' state initializes, the recording is stopped via ARI using the [ARI Wrapper](ari.md) module. 

### save

When save is called, a promise is returned that will be resolved when a RecordingSaved event is raised by the finite state machine. Any error that occurs will instead cause the promise to be rejected with that error.

If the finite state machine is in a 'recordingFinished' state, the finite state machine will then transition to a 'savingRecording' state. Once that state is initialized, a new message model instance is created with the recording and attached to the INBOX folder. The message is then saved in the database with the [Data](data.md) module and MWI counts for the mailbox are updated via ARI using the [Notify](notify.md) module. The finite state machine then transitions to a 'done' state where no more inputs can be processed and a RecordingSaved event is raised by the finite state machine so that the promise returned by the save method can be resolved.

## test/

Tests can be run using the following command:

```bash
$ grunt mochaTest
```

The unit tests for the voicemail mailbox module are separated into separate files.

### messages.js

All mocha unit tests related to message sequences created with lib/helpers/messages.js can be found here.

### reader.js

All mocha unit tests related to the mailbox reader can be found here.

### writer.js

All mocha unit tests related to the mailbox writer can be found here.

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

Voicemail Mailbox depends on the following voicemail modules:

- [ARI Wrapper](ari.md)
- [Config](config.md)
- [Data](data.md)
- [Prompt](prompt.md)
- [Notify](notify.md)
- [Logging](logging.md)
