# Voicemail Notify Documentation

## lib/notify.js

[Voicemail Notify](https://github.com/asterisk/node-voicemail-notify) is a module responsible for voicemail related notifications.

When called with all required dependencies, it exposes a create method that allows the creation of a notifier object. The create method accepts a mailbox and a message associated with a notification. The message can be a new message for the mailbox, or a message that has been listened to in the mailbox, or a message that was deleted from the mailbox.

The notifier object returned by the create method first creates an update function to update MWI counts for the given mailbox that accepts a read and an unread count. This function will be passed along to the [Data](data.md) module and called by it with the current counts for the given mailbox. This function uses the [Config](config.md) and the [ARI Wrapper](ari.md) modules to connect to ARI, get a client, and update the MWI counts for the given mailbox.

The notifier object then returns an object with the following methods:

### newMessage

When newMessage is called, the [Data](data.md) module is used to update the mailbox in the database to reflect the fact that a new message exists. The MWI update function created by the notifier is passed along so that MWI counts can be updated in ARI.

### messageRead

When messageRead is called, the [Data](data.md) module is used to update the mailbox in the database to reflect the fact that a message has been listened to. The MWI update function created by the notifier is passed along so that MWI counts can be updated in ARI.

### messageDeleted

When messageDeleted is called, the [Data](data.md) module is used to update the mailbox in the database to reflect the fact that a message was deleted. The MWI update function created by the notifier is passed along so that MWI counts can be updated in ARI.

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

Voicemail Notify depends on the following voicemail modules:

- [Data](data.md)
- [Config](config.md)
- [ARI Wrapper](ari.md)
