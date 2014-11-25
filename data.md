# Voicemail Data Access Layer Documentation

## lib/

### db.js

The [Voicemail Data Access Layer](https://github.com/asterisk/node-voicemail-data) module is exported in this file. When called with a database configuration object and all required dependencies, an object keyed by repository name is returned given the connection string found in the database configuration object. The object is cached using the connection string so that asking for repositories for a given connection string more than once will yield the same object. The object returned by calling the module export with a database config contains the following keys:

- context
- mailbox
- folder
- message
- contextConfig
- mailboxConfig

### helpers/

#### common.js

This file contains methods used by all repositories. These methods allow an abstract, repository independent, way of interacting with the tables that make up the data access layer. These methods rely on an instance of a provider as well as a [node-sql](https://github.com/brianc/node-sql) representation of the table to be operated on to collect required meta-data. The following methods are exposed:

- optionalArgument (returns the first argument to contain an expected argument type - helper method for optional arguments)
- populateFields (given fields retrieve from a database, attaches the fields to the instance if the instance defines those fields)
- createTable (creates a table using the [node-sql](https://github.com/brianc/node-sql) table definition)
- createIndex (creates a index using the given definitions)
- get (returns a single object instance using a given where clause)
- find (Returns a sequence of objects for a given query)
- count (Returns the row count for a given query)
- save (writes a row to the database using a given instance)
- remove (removes a row from the database using a given instance)

This helper module is also responsible for converting data from the database to object instances and back. The following methods are used for this purpose:

- instanceFromRow (converts a database row to an instance of a repository model)
- getFields (determines which fields should be stored in the database given a repository model instance and a [node-sql](https://github.com/brianc/node-sql) table definition as well as converts reference types to foreign keys)

### providers/

#### postgres.js

This file contains PostgreSQL specific implementation details. The following methods are exposed:

##### runQuery

Runs a single query wrapped in a transaction.

##### beginTransaction

Begins a transaction and returns an object that can be used to progress through the transaction. The object contains the following methods:

- runQuery
- commit
- rollback

##### forUpdate

Adds 'for update' to the query text to enable row locking.

##### autoIncrement

Converts primary key query text to be of type serial to enable auto incrementing IDs.

##### convertDataFromStorage

Converts the date that was stored without a time zone to a [moment](https://github.com/moment/moment) instance in UTC format.

##### converDataForStorage

Leaves the date untouched. PostgreSQL will simply store the data without a time zone. We will later read it as a UTC formatted date.

##### getDateType

Returns the date type to be used for PostgreSQL. In this case we are using a timestamp without a time zone.

#### sqlite.js

This file contains sqlite specific implementation details. The following methods are exposed:

##### runQuery

Runs a single query wrapped in a transaction.

##### beginTransaction

Begins a transaction and returns an object that can be used to progress through the transaction. Since sqlite does not support row level locking, a BEGIN IMMEDIATE statement is used to begin a transaction level lock immediately instead of using deferred locking. The object contains the following methods:

- runQuery
- commit
- rollback

##### forUpdate

Returns the query as is since sqlite does not use row level locking.

##### autoIncrement

Returns the create statement as is since sqlite will automatically auto increment integer primary keys.

##### convertDataFromStorage

Reads the date as a unix timestamp and converts it to a [moment](https://github.com/moment/moment) instance in UTC format.

##### converDataForStorage

Converts the date to a unix timestamp using [moment](https://github.com/moment/moment).

##### getDateType

Returns the date type to be used for sqlite. In this case we are using an integer since we will be storing dates as unix timestamps.

### repositories/

#### context.js

This file exposes a context repository. When called with a database config, an instance of the configured provider is created, a table definition is then created using [node-sql](https://github.com/brianc/node-sql) and an instance of a SQL generator for the configured provider is created to allow SQL statement building.

The repository is then composed using [compose](https://github.com/kriszyp/compose) to allow the configured provider to override the methods found in helpers/common.js. The returned repository exposes the following methods:

##### createTable

Creates the context table in the database using helpers/common.js.

##### createdIndexes

Creates indexes for the context table using helpers/common.js.

##### create

Constructs and returns a context instance. This will return the following object:

```JavaScript
{
  domain: '...',
  getId: function() {
    ...
  }
}
```

##### get

Returns a context instance from the database given a domain using helpers/common.js.

##### save

Saves a given context instance to the database using helpers/common.js.

##### remove

Removes a given context instance from the database using helpers/common.js.

#### contextConfig.js

This file exposes a context configuration repository. When called with a database config, an instance of the configured provider is created, a table definition is then created using [node-sql](https://github.com/brianc/node-sql) and an instance of a SQL generator for the configured provider is created to allow SQL statement building.

The repository is then composed using [compose](https://github.com/kriszyp/compose) to allow the configured provider to override the methods found in helpers/common.js. The returned repository exposes the following methods:

##### createTable

Creates the context config table in the database using helpers/common.js.

##### createdIndexes

Creates indexes for the context config table using helpers/common.js.

##### create

Constructs and returns a context config instance. This will return the following object:

```JavaScript
{
  key: undefined,
  value: undefined,
  getId: function() {
    ...
  },
  getContext: function() {
    ...
  }
}
```

This object can then be populated using a database row.

##### all

Returns all context config instances from the database given a context instance using helpers/common.js.

##### save

Saves a given context config instance to the database using helpers/common.js.

##### remove

Removes a given context config instance from the database using helpers/common.js.

#### folder.js

This file exposes a folder repository. When called with a database config, an instance of the configured provider is created, a table definition is then created using [node-sql](https://github.com/brianc/node-sql) and an instance of a SQL generator for the configured provider is created to allow SQL statement building.

The repository is then composed using [compose](https://github.com/kriszyp/compose) to allow the configured provider to override the methods found in helpers/common.js. The returned repository exposes the following methods:

##### createTable

Creates the folder table in the database using helpers/common.js.

##### createdIndexes

Creates indexes for the folder table using helpers/common.js.

##### create

Constructs and returns a folder instance. This will return the following object:

```JavaScript
{
  name: undefined,
  recording: undefined,
  dtmf: undefined,
  getId: function() {
    ...
  }
}
```

This object can then be populated using a database row.

##### all

Returns all folder instances from the database using helpers/common.js. The object returns will be keyed by DTMF key associated with a given folder.

##### save

Saves a given folder instance to the database using helpers/common.js.

##### remove

Removes a given folder instance from the database using helpers/common.js.

#### mailbox.js

This file exposes a mailbox repository. When called with a database config, an instance of the configured provider is created, a table definition is then created using [node-sql](https://github.com/brianc/node-sql) and an instance of a SQL generator for the configured provider is created to allow SQL statement building.

The repository is then composed using [compose](https://github.com/kriszyp/compose) to allow the configured provider to override the methods found in helpers/common.js. The returned repository exposes the following methods:

##### createTable

Creates the mailbox table in the database using helpers/common.js.

##### createdIndexes

Creates indexes for the mailbox table using helpers/common.js.

##### create

Constructs and returns a mailbox instance. This will return the following object:

```JavaScript
{
  mailboxNumber: number,
  mailboxName: undefined,
  password: undefined,
  name: undefined,
  email: undefined,
  read: undefined,
  unread: undefined,
  greetingBusy: undefined,
  greetingAway: undefined,
  greetingName: undefined,
  getId: function() {
   ...
  },
  getContext: function() {
   ...
  }
}
```

This object can then be populated using a database row.

##### get

Returns a single mailbox instance given a mailbox number and context instance using helpers/common.js.

##### save

Saves a given mailbox instance to the database using helpers/common.js.

##### remove

Removes a given mailbox instance from the database using helpers/common.js.

##### newMessage

Updates the unread count of the given mailbox using the given mwi updater method. This uses the provider specific beginTransaction and the methods it exposes to atomically update the unread count on the mailbox and update MWI counts via ARI using the provided mwi updater method. This method will increment the unread count for the given mailbox.

##### readMessage

Updates the read/unread count of the given mailbox using the given read status mwi updater method. This uses the provider specific beginTransaction and the methods it exposes to atomically update the read/unread count on the mailbox and update MWI counts via ARI using the provided mwi updater method. This method will increment the read count and decrement the unread count for the given mailbox.

##### deletedMessage

Updates the read/unread count of the given mailbox using the given mwi updater method. This uses the provider specific beginTransaction and the methods it exposes to atomically update the read/unread count on the mailbox and update MWI counts via ARI using the provided mwi updater method. If the message was read, this method will decrement the read count for the given mailbox. If the message was not read, it will decrement the unread count instead.

#### mailboxConfig.js

This file exposes a mailbox configuration repository. When called with a database config, an instance of the configured provider is created, a table definition is then created using [node-sql](https://github.com/brianc/node-sql) and an instance of a SQL generator for the configured provider is created to allow SQL statement building.

The repository is then composed using [compose](https://github.com/kriszyp/compose) to allow the configured provider to override the methods found in helpers/common.js. The returned repository exposes the following methods:

##### createTable

Creates the mailbox config table in the database using helpers/common.js.

##### createdIndexes

Creates indexes for the mailbox config table using helpers/common.js.

##### create

Constructs and returns a mailbox config instance. This will return the following object:

```JavaScript
{
  key: undefined,
  value: undefined,
  getId: function() {
    ...
  },
  getMailbox: function() {
    ...
  }
}
```

This object can then be populated using a database row.

##### all

Returns all mailbox config instances from the database given a mailbox instance using helpers/common.js.

##### save

Saves a given mailbox config instance to the database using helpers/common.js.

##### remove

Removes a given mailbox config instance from the database using helpers/common.js.

#### message.js

This file exposes a message repository. When called with a database config, an instance of the configured provider is created, a table definition is then created using [node-sql](https://github.com/brianc/node-sql) and an instance of a SQL generator for the configured provider is created to allow SQL statement building.

The repository is then composed using [compose](https://github.com/kriszyp/compose) to allow the configured provider to override the methods found in helpers/common.js. The returned repository exposes the following methods:

##### createTable

Creates the message table in the database using helpers/common.js.

##### createdIndexes

Creates indexes for the message table using helpers/common.js.

##### create

Constructs and returns a message instance. This will return the following object:

```JavaScript
{
  date: undefined,
  read: undefined,
  originalMailbox: undefined,
  callerId: undefined,
  duration: undefined,
  recording: undefined,
  getId: function() {
    ...
  },
  getMailbox: function() {
    ...
  },
  getFolder: function() {
    ...
  },
  init: function() {
    ...
  },
  markAsRead: function() {
    ...
  }
}
```

This object can then be populated using a database row.

##### all

Returns all message instances from the database given a mailbox and folder instance using helpers/common.js. This method uses concurrent promises to batch up fetching messages in groups of fifty.

##### get

Uses a given message instance to return the latest values from the database. Private fields remain unchanged.

##### markAsRead

Marks the message as read in the database. This uses the provider specific beginTransaction and the methods it exposes to atomically fetch the given message to ensure it as not been marked as read already. Once this has been done, a boolean flag is returned to indicate whether or not the message was updated. This can then be used to update the mailbox MWI counts using the mailbox repository.

##### latest

Returns the latest messages for the given mailbox and folder using a [moment](https://github.com/moment/moment) date as the current latest message date using helpers/common.js.

##### save

Saves a given message instance to the database using helpers/common.js.

##### changeFolder

Changes the folder the given message belongs to using the given folder instance.

##### remove

Removes a given message instance from the database. This uses the provider specific beginTransaction and the methods it exposes to atomically fetch the given message before deleting it from the database. Once this has been done, the message instance is returned. This can then be used to update the mailbox MWI counts using the mailbox repository.

## Schema

The schema for the voicemail data access layer is made up of the following five tables:

### Context

| NAME             | TYPE                        | USAGE                                 |
| :--------------- | :-------------------------- | :------------------------------------ |
| ID               | SERIAL                      | Primary key                           |
| DOMAIN           | VARCHAR(254)                | The context's domain name (email.com) |

### Context Config

| NAME             | TYPE                        | USAGE                                 |
| :--------------- | :-------------------------- | :------------------------------------ |
| ID               | SERIAL                      | Primary key                           |
| CONTEXT_ID       | INTEGER                     | Foreign key to CONTEXT table          |
| KEY              | VARCHAR(100)                | Configuration key                     |
| VALUE            | VARCHAR(100)                | Configuration value                   |

### Folder

| NAME             | TYPE                        | USAGE                                 |
| :--------------- | :-------------------------- | :------------------------------------ |
| ID               | SERIAL                      | Primary key                           |
| NAME             | VARCHAR(25)                 | The name of the folder                |
| RECORDING        | VARCHAR(100)                | The path to the folder recording      |
| DTMF             | INTEGER                     | DTMF key associated with the folder   |

### Mailbox

| NAME             | TYPE                        | USAGE                                             |
| :--------------- | :-------------------------- | :------------------------------------------------ |
| ID               | SERIAL                      | Primary key                                       |
| MAILBOX_NUMBER   | INTEGER                     | The mailbox number                                |
| CONTEXT_ID       | INTEGER                     | Foreign key to CONTEXT table                      |
| PASSWORD         | VARCHAR(100)                | The mailbox password                              |
| NAME             | VARCHAR(100)                | The mailbox owner's name                          |
| EMAIL            | VARCHAR(256)                | The mailbox owner's email                         |
| GREETING_AWAY    | VARCHAR(100)                | Path to recording for when owner is unavailable   |
| GREETING_BUSY    | VARCHAR(100)                | Path to recording for when owner is busy          |
| GREETING_NAME    | VARCHAR(100)                | Path to recording of the mailbox owner's name     |
| MAILBOX_NAME     | VARCHAR(255)                | The mailbox's name for use in updating MWI counts |
| READ             | INTEGER                     | Number of read messages in the mailbox            |
| UNREAD           | INTEGER                     | Number of unread messages in the mailbox          |

### Mailbox Config

| NAME             | TYPE                        | USAGE                                 |
| :--------------- | :-------------------------- | :------------------------------------ |
| ID               | SERIAL                      | Primary key                           |
| MAILBOX_ID       | INTEGER                     | Foreign key to MAILBOX table          |
| KEY              | VARCHAR(100)                | Configuration key                     |
| VALUE            | VARCHAR(100)                | Configuration value                   |

### Message

| NAME             | TYPE                        | USAGE                                             |
| :--------------- | :-------------------------- | :------------------------------------------------ |
| ID               | SERIAL                      | Primary key                                       |
| MAILBOX_ID       | INTEGER                     | Foreign key to MAILBOX table                      |
| RECORDING        | VARCHAR(100)                | Path to the message recording                     |
| READ             | CHAR(1)                     | 'Y' if message has been read, 'N' otherwise       |
| DATE             | TIMESTAMP                   | Timestamp representing when the message was left  |
| ORIGINAL_MAILBOX | INTEGER                     | Mailbox number of original mailbox (forwarded)    |
| CALLER_ID        | VARCHAR(100)                | Caller ID of the person who left the message      |
| DURATION         | VARCHAR(100)                | Duration in seconds of the message left           |
| FOLDER_ID        | INTEGER                     | Foreign key to the FOLDER table                   |

## test/

Tests can be run using the following command:

```bash
$ grunt mochaTest
```

The unit tests for the data access layer are separated into separate files by the repository methods they cover.

### helpers/

#### common.js

Common helper methods for test setup and teardown can be found here.

#### fixtures.json

Fixtures for tests can be found here.

### context.js

All mocha unit tests related to context methods can be found here.

### contextConfig.js

All mocha unit tests related to context config methods can be found here.

### folder.js

All mocha unit tests related to folder methods can be found here.

### mailbox.js

All mocha unit tests related to mailbox methods can be found here.

### mailboxConfig.js

All mocha unit tests related to mailbox config methods can be found here.

### message.js

All mocha unit tests related to message methods can be found here.

## Gruntfile.js

Grunt tasks for running a linter, unit tests, and code coverage can be found here. The linter and unit tests can be run using the following command:

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

Voicemail Data Access Layer depends on the following voicemail modules:

- [Logging](logging.md)
