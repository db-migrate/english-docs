# Programable API

This API is intended for the usage with db-migrate as a module and consists out
of several functions.

## Using db-migrate as module

You can always find example on how to use the API in our
example project:

https://github.com/db-migrate/api-examples

To get an instance of db-migrate you need to call it in your application like the following:

```javascript
var DBMigrate = require('db-migrate');
var assert = require('assert');

//getting an instance of dbmigrate
var dbmigrate = DBMigrate.getInstance(true);

//execute any of the API methods
dbmigrate.reset()
.then( () => dbmigrate.up() );
```

## Handle the onComplete callback outside of db-migrate

If you want to handle the onComplete callback in your own project you can do
this like this:

```javascript
function specialCallback(migrator, originalError) {
  migrator.driver.close(function(err) {
    assert.ifError(originalErr);
    assert.ifError(err);
    log.info('Done');
  });
}

//specify an own callback, to handle errors on your side of the application.
var dbmigrate_2 = DBMigrate.getInstance(true, specialCallback);
```

## Start the CLI mode

If you need to use the CLI mode and just made some customizations
you can do this via the following example:

```javascript
//get an instance and call the standard runtime behavior
var dbmigrate_3 = DBMigrate.getInstance();
dbmigrate_3.run();
```

### getInstance(isModule, options, callback)

Get an instance of the db-migrate API.

__Arguments__

* isModule - set true to return the API as a node module
* options - hash of various options
* callback - custom callback

__Options__

* cwd - working directory (default: process.cwd)
* config
     - string - location of the database.json file
     - object - hash of [configuration](https://umigrate.readthedocs.org/projects/db-migrate/en/latest/Getting%20Started/configuration/) options
* cmdOptions - hash of CMD options from [Basic Usage](https://db-migrate.readthedocs.io/en/latest/Getting%20Started/installation/)
* env - the environment to run the migrations under
* throwUncatched - Throw an error instead of calling `process.exit(1)`

# Programable API

### registerAPIHook([callback])

Register all API hooks and initializes them. This needs to be executed
before `run` is being executed.

__Arguments__

* callback - custom callback

__Examples__

```javascript
var dbm = dbmigrate.getInstance(true);
dbm.registerAPIHook()
.then(function() {

  dbm.run();
});
```

## run()

Executes the default post initialization CLI behavior.

__Examples__

```javascript
var dbm = dbmigrate.getInstance(true);
dbm.run();
```

## up([[specification][count], [scope], [callback]])

Migrates in upwards direction. This is equal to the CLI UP.

__Arguments__

* specification - Migration to migrate up to needs to start with this string
* count - Number of migrations to be executed.
* scope - Scope to be used or omitted.
* callback - custom callback, omitted if using Promises

__Examples__

```javascript
var dbm = dbmigrate.getInstance(true);
dbm.up(12)
.then(function() {

  console.log('successfully migrated 12 migrations up');
  return;
});
```

## down([[specification][count], [scope], [callback]])

Migrates in downwards direction. This is equal to the CLI DOWN.

__Arguments__

* specification - Migration to migrate down to needs to start with this string
* count - Number of migrations to be executed.
* scope - Scope to be used or omitted.
* callback - custom callback, omitted if using Promises

__Examples__

```javascript
var dbm = dbmigrate.getInstance(true);
dbm.down(12)
.then(function() {

  console.log('successfully migrated 12 migrations down');
  return;
});
```

## sync([[specification][count], [scope] [callback]])

Migrates to the specified migration and automatically detects if it needs to
migrate up or down. This is equal to the CLI SYNC.

__Arguments__

* specification - Migration which is the destination and which needs to start
with this string
* scope - Scope to be used or omitted.
* callback - custom callback, omitted if using Promises

__Examples__

```javascript
var dbm = dbmigrate.getInstance(true);
dbm.sync('20150207135259')
.then(function() {

  console.log('successfully migrated 12 migrations up');
  return;
});
```

## reset([[scope], [callback]])

Migrates all currently executed migrations down.

__Arguments__

* scope - Scope to be used or omitted.
* callback - custom callback, omitted if using Promises
 a Hook.
__Examples__

```javascript
var dbm = dbmigrate.getInstance(true);
dbm.reset()
.then(function() {

  console.log('successfully resetted all migrations!');
  return;
});
```

## silence(isSilent)

Silences or unsilences logs.

__Arguments__

* isSilent - Boolean that silences if true is passed.

__Examples__

```javascript
var dbm = dbmigrate.getInstance(true);
dbm.silence(true);
dbm.reset();
```

## transition()

Starts the transmission helper for the dbmigrate protocol migration.

__Examples__

```javascript
var dbm = dbmigrate.getInstance(true);
dbm.transition();
```

## create(migrationName[, [scope], [callback]])

Creates a new migration from a template.

__Arguments__

* migrationName - The name for the new migration.
* Scope - The scope to be used, can be omitted
* callback - custom callback, omitted if using Promises

## createDatabase(dbname[, callback])

Creates a database with the name specified.

__Arguments__

* dbname - The name of the database.
* callback - custom callback, omitted if using Promises

## dropDatabase(dbname[, callback])

Deletes the specified database.

__Arguments__

* dbname - The name of the database.
* callback - custom callback, omitted if using Promises
