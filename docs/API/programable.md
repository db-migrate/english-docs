# Programable API

db-migrate can be used as a module, for example to migrate on application
start or in tests. Examples are in the
[api-examples project](https://github.com/db-migrate/api-examples).

```javascript
const DBMigrate = require('db-migrate');

const dbmigrate = DBMigrate.getInstance(true, {
  env: 'test',
  config: {
    test: { driver: 'sqlite3', filename: ':memory:' }
  }
});

await dbmigrate.reset();
await dbmigrate.up();
```

Every method running migrations returns a promise and takes an optional
callback as last argument. A failure rejects with the error described in
[Failures and recovery](../Guides/failures and recovery.md#error-output).

Create a new instance for every run, the instance keeps the options of the
previous call, like a count or a scope.

## getInstance(isModule, [options], [callback])

Get an instance of the db-migrate API.

__Arguments__

* isModule - `true` to use db-migrate as a module. Otherwise the command line
  of the process is parsed and the instance is meant for `run()`.
* options - see below
* callback - a custom [onComplete callback](#custom-oncomplete-callback)

__Options__

* cwd - working directory (default: `process.cwd()`), the base of the default
  config file, migrations directory and plugins
* config
     - string - location of the config file
     - object - the [configuration](../Getting Started/configuration.md)
       itself, environments as keys
* env - the environment to use
* cmdOptions - the [options](../Getting Started/commands.md#options) of the
  command line, with their long names, e.g.
  `{ 'migrations-dir': 'db/migrations', 'v2-file': true }`
* throwUncatched - do not register handlers for uncaught exceptions and
  unhandled rejections, which log the error and call `process.exit(1)`
* noPlugins - `true` to not load the [plugins](../Getting Started/plugins.md)
  of the `package.json`
* plugins - plugins to register in addition, an object of hook names, each
  with an array of plugins

The [rc configs](../Getting Started/configuration.md#rc-configs) are applied
in module mode as well. The arguments of the command line of your process are
left alone since db-migrate 1.1.0, before they were applied as options of
db-migrate.

__Properties__

* version - the version of db-migrate
* dataType - the [data types](generic datatypes.md)
* config - the loaded configuration

## up([specification | count], [scope], [callback])

Migrates up. This is equal to the CLI `up`.

__Arguments__

* specification - a string, run the pending migrations up to the one starting
  with this string
* count - a number, the maximum number of migrations to run
* scope - the [scope](../Getting Started/commands.md#scoping) to use
* callback - custom callback, omitted if using Promises

__Examples__

```javascript
await dbmigrate.up();
await dbmigrate.up(12);
await dbmigrate.up('20150207135259');
await dbmigrate.up(1, 'test');
```

## down([specification | count], [scope], [callback])

Migrates down. This is equal to the CLI `down`, without arguments only the
last migration is reverted.

__Arguments__

* specification - a string, revert every migration executed after the one
  starting with this string
* count - a number, the maximum number of migrations to revert
* scope - the scope to use
* callback - custom callback, omitted if using Promises

## reset([scope], [callback])

Reverts all executed migrations of the scope.

## sync(specification, [scope], [callback])

Migrates up or down to the migration starting with `specification`. This is
equal to the CLI `sync`.

```javascript
await dbmigrate.sync('20150207135259');
```

## check([count], [scope], [callback])

Resolves with the pending migrations, objects with their `name`, without
running them. This is equal to the CLI `check`. The count has no effect, pass
`null` to give a scope only.

```javascript
const pending = await dbmigrate.check();
console.log(pending.map(m => m.name));

await dbmigrate.check(null, 'test');
```

## fix([specification | count], [scope], [callback])

Rebuilds the schema learned from v2 migrations, see the CLI
[fix](../Getting Started/commands.md#fix).

## create(migrationName, [scope], [callback])

Creates a new migration from a template. Choose the template with
`cmdOptions` or `setConfigParam`, e.g. `setConfigParam('v2-file', true)`.

__Arguments__

* migrationName - the name of the new migration
* scope - the scope to create it in
* callback - custom callback, omitted if using Promises

## createDatabase(dbname, [callback]) and dropDatabase(dbname, [callback])

Create or drop a database, like `db:create` and `db:drop`.

The promise resolves once the database is created or dropped. Before
db-migrate 1.1.0 it resolved early and the process exited afterwards.

## seed, undoSeed and resetSeed

Seeders are not supported in 1.0, these methods reject with
`Seeders are not supported by db-migrate 1.0`.

## silence(isSilent)

Silences or unsilences the log output of db-migrate.

```javascript
dbmigrate.silence(true);
```

## setConfigParam(param, value)

Sets an option, like on the command line, before the next call:

```javascript
dbmigrate.setConfigParam('force-exit', true);
```

## run()

Executes the command line behavior, the command and options are taken from
the arguments of the process. This is what the `db-migrate` binary does:

```javascript
const dbmigrate = DBMigrate.getInstance();
dbmigrate.registerAPIHook().then(() => dbmigrate.run());
```

## registerAPIHook([callback])

Registers the API functions added by plugins on the instance, see
[Writing plugins](../Developers/writing plugins.md#initapiaddfunctionhook-every-plugin).
Returns a promise.

## Custom onComplete callback

When a run finished, db-migrate calls its onComplete function, which closes
the connection and logs `Done`. Replace it to handle the end of a run on your
own, with the callback of `getInstance` or `setCustomCallback(callback)`;
`setDefaultCallback()` restores the default.

```javascript
function onComplete(migrator, internals, originalErr, results) {
  return new Promise((resolve, reject) => {
    migrator.driver.close(err => {
      if (originalErr || err) return reject(originalErr || err);
      resolve(results);
    });
  });
}

const dbmigrate = DBMigrate.getInstance(true, {}, onComplete);
```

It has to close the connection of `migrator.driver`. Its return value is what
the promise of the method resolves with.
