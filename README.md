# Voicemail Documentation

This repository documents the design and implementation of the Voicemail and Voicemail Main applications. These applications were built using Node.js and ARI to interface with Asterisk.

- Voicemail can be found [here](https://github.com/asterisk/node-voicemail)
- Voicemail Main can be found [here](https://github.com/asterisk/node-voicemail-main)

## Goals

The goals of Voicemail was to implement a modular and configurable replacement for Asterisk Voicemail using ARI and Node.js that could be easily modified to improve and add to core features while also allowing individuals who desire more control over the behavior of voicemail to fork modules or the entire application if they so choose.

Node.js was selected for it's asynchronous and events API which pair very well with ARI asynchronous HTTP API and WebSocket architecture. Node.js also enables a growing community of developers to built on voicemail using JavaScript.

## Design

Part of designing Voicemail involved building a [prototype](https://github.com/asterisk/node-voicemail-js) to test out some early ideas. Thanks to feedback from the Asterisk community and lessons learned while building the prototype, the following design goals were reached:

- Finite state machines that drive all business logic.
- Modules implemented with a finite state machine expose an API that abstract out their implementation.
- Separate applications for Voicemail and Voicemail Main.
- Modules separated into repositories and loaded via git url dependencies using NPM.
- Modules are responsible for registering their own event handlers and cleaning up upon a channel exiting the application.
- Promises over callbacks for asynchronous APIs.
- Functions returning object literals with closures to support private data as opposed to multiple layers of prototypical objects.
- Data layer organized by methods operating on given tables instead of grouping all methods together in an implementation of a large interface.
- Data layer that dynamically build objects from database records using naming conventions as opposed to hard-coding fields.

## Implementation

Voicemail was implemented over two application modules, two business logic modules, and seven helper modules used by the business logic modules. A brief description of each module can be found below along with links to more detailed documentation for each module.

### Modules

#### Applications

These modules are responsible for bootstrapping, establishing a connection to Asterisk and handling incoming channels. They are also responsible for loading module dependencies which are then passed to lower level modules.

- [Voicemail](voicemail.md) is an application responsible for users leaving messages.
- [Voicemail Main](voicemail-main.md) is an application responsible for users listening to messages in their mailbox.

#### Business logic driving the applications

These modules are responsible for driving the applications through defined business logic. These are implemented as finite state machines. Once they receive an incoming channel, they control the channel until it exits the application. The calling applications no longer need to interact with the channel at this point. These modules are responsible for all state and cleanup involving incoming channels.

- [Voicemail FSM](voicemail-fsm.md) is a finite state machine responsible for implementing all business logic involved in leaving messages.
- [Voicemail Main FSM](voicemail-main-fsm.md) is a finite state machine responsible for implementing all business logic involved in listening to messages.

#### Helper modules

- [Data](data.md) is a data access layer for the applications.
- [Auth](auth.md) is an authentication helper for accessing a given mailbox.
- [Prompt](prompt.md) is a helper responsible for playing sequences of sounds and allowing the sequence to be stopped at any point.
- [Mailbox](mailbox.md) is a helper to interact with mailboxes. It exposes both a writer (for leaving messages) and a reader (for listening to messages).
- [Notify](notify.md) is a notification helper for interacting with MWI/sending emails, et cetera.
- [Config](config.md) is a helper for fetching application and mailbox level configuration.
- [ARI Wrapper](ari.md) is a helper that connects to a single Stasis application via ARI and acts as a client singleton.

### Libraries

The following libraries were used in the implementation of Voicemail:

- [q](https://github.com/kriskowal/q) promises
- [machina](https://github.com/ifandelse/machina.js) finite state machine builder
- [sql](https://github.com/brianc/node-sql) SQL statement builder
- [pg](https://github.com/brianc/node-postgres) PostgreSQL client
- [sqlite3](https://github.com/mapbox/node-sqlite3) sqlite client
- [compose](https://github.com/kriszyp/compose) object composition for overriding data access behavior
- [case](https://github.com/nbubna/Case) string case manipulation for converting from database column names to object properties and back
- [moment](https://github.com/moment/moment) date time manipulation
- [grunt](https://github.com/gruntjs/grunt) build tool
- [mocha](https://github.com/mochajs/mocha) test runner
- [mockery](https://github.com/mfncooper/mockery) test mocking
- [istanbul](https://github.com/gotwarlost/istanbul) test coverage
- [grunt-mocha-test](https://github.com/pghalliday/grunt-mocha-test) grunt mocha plugin
- [grunt-contrib-jshint](https://github.com/gruntjs/grunt-contrib-jshint) grunt jshint plugin for linting
- [grunt-mocha-istanbul](https://github.com/pocesar/grunt-mocha-istanbul) grunt mocha reporter for test coverage with istanbul
