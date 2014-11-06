# Voicemail Application Documentation

[Voicemail](https://github.com/asterisk/node-voicemail) is the application responsible for allowing users to leave messages. Its main responsibility is to bootstrap the application by loading the appropriate dependencies.

After all dependencies have been loaded, it connects to Asterisk via ARI and establishes a WebSocket connection so that the voicemail Stasis application is ready for incoming requests.

It then registers an event handler for a StasisStart event. Once a channel enters into the application via Stasis, a [finite state machine](https://github.com/asterisk/node-voicemail-fsm) to drive voicemail business logic is spun up and the StasisStart event as well as an object representing the incoming channel are passed over to it at which point the finite state machines takes control of the application for the given channel.

The application can be started by running the following:

```bash
$ node app.js
```

or by loading it as a module and starting it manually:

```JavaScript
var voicemail = require('voicemail');

voicemail.create();
```

This allows the application to be run from a terminal or loaded as part of a larger application.
