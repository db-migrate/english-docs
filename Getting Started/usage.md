# Usage of db-migrate

To use db-migrate, you call it via the command line. When entering only the
command without paramaters you will see something like this:

```
Usage: db-migrate [up|down|reset|create|db] [[dbname/]migrationName|all] [options]

Down migrations are run in reverse run order, so migrationName is ignored for down migrations.
Use the --count option to control how many down migrations are run (default is 1).

Options:
  --env, -e                   The environment to run the migrations under.    [default: "dev"]
  --migrations-dir, -m        The directory containing your migration files.  [default: "./migrations"]
  --count, -c                 Max number of migrations to run.
  --dry-run                   Prints the SQL but doesn't run it.              [boolean]
  --verbose, -v               Verbose mode.                                   [default: false]
  --config                    Location of the database.json file.             [default: "./database.json"]
  --force-exit                Call system.exit() after migration run          [default: false]
  --sql-file                  Create sql files for up and down.               [default: false]
  --coffee-file               Create a coffeescript migration file            [default: false]
  --migration-table           Set the name of the migration table.
  --table, --migration-table                                                  [default: "migrations"]
```

## Creating Migrations

To create a migration, execute `db-migrate create` with a title. `node-db-migrate` will create a node module within `./migrations/` which contains the following two exports:

```javascript
exports.up = function (db, callback) {
  callback();
};

exports.down = function (db, callback) {
  callback();
};
```

All you have to do is populate these, invoking `callback()` when complete, and you are ready to migrate!

For example:

    $ db-migrate create add-pets
    $ db-migrate create add-owners

The first call creates `./migrations/20111219120000-add-pets.js`, which we can populate:

```javascript
exports.up = function (db, callback) {
  db.createTable('pets', {
    id: { type: 'int', primaryKey: true },
    name: 'string'
  }, callback);
};

exports.down = function (db, callback) {
  db.dropTable('pets', callback);
};
```

The second creates `./migrations/20111219120005-add-owners.js`, which we can populate:

```javascript
exports.up = function (db, callback) {
  db.createTable('owners', {
    id: { type: 'int', primaryKey: true },
    name: 'string'
  }, callback);
};

exports.down = function (db, callback) {
  db.dropTable('owners', callback);
};
```

Executing multiple statements against the database within a single migration requires a bit more care. You can either nest the migrations like:

```javascript
exports.up = function (db, callback) {
  db.createTable('pets', {
    id: { type: 'int', primaryKey: true },
    name: 'string'
  }, createOwners);

  function createOwners(err) {
    if (err) { callback(err); return; }
    db.createTable('owners', {
      id: { type: 'int', primaryKey: true },
      name: 'string'
    }, callback);
  }
};

exports.down = function (db, callback) {
  db.dropTable('pets', function(err) {
    if (err) { callback(err); return; }
    db.dropTable('owners', callback);
  });
};
```

or use the async library to simplify things a bit, such as:

```javascript
var async = require('async');

exports.up = function (db, callback) {
  async.series([
    db.createTable.bind(db, 'pets', {
      id: { type: 'int', primaryKey: true },
      name: 'string'
    }),
    db.createTable.bind(db, 'owners', {
      id: { type: 'int', primaryKey: true },
      name: 'string'
    });
  ], callback);
};

exports.down = function (db, callback) {
  async.series([
    db.dropTable.bind(db, 'pets'),
    db.dropTable.bind(db, 'owners')
  ], callback);
};
```

### Using files for sqls

If you prefer to use sql files for your up and down statements, you can use the `--sql-file` option to automatically generate these files and the javascript code that load them.

For example:

    $ db-migrate create add-people --sql-file

This call creates 3 files:

```
./migrations/20111219120000-add-people.js
./migrations/sqls/20111219120000-add-people-up.sql
./migrations/sqls/20111219120000-add-people-down.sql
```

The sql files will have the following content:
```sql
/* Replace with your SQL commands */
```

And the javascript file with the following code that load these sql files:

```javascript
dbm = dbm || require('db-migrate');
var type = dbm.dataType;
var fs = require('fs');
var path = require('path');

exports.up = function(db, callback) {
  var filePath = path.join(__dirname + '/sqls/20111219120000-add-people-up.sql');
  fs.readFile(filePath, {encoding: 'utf-8'}, function(err,data){
    if (err) return console.log(err);
    db.runSql(data, function(err) {
      if (err) return console.log(err);
      callback();
    });
  });
};

exports.down = function(db, callback) {
  var filePath = path.join(__dirname + '/sqls/20111219120000-add-people-down.sql');
  fs.readFile(filePath, {encoding: 'utf-8'}, function(err,data){
    if (err) return console.log(err);
    db.runSql(data, function(err) {
      if (err) return console.log(err);
      callback();
    });
  });
};
```

** Making it as default **

To not need to always specify the `sql-file` option in your `db-migrate create` commands, you can set a property in your `database.json` as follows:

```
{
    "dev": {
      "host": "localhost",
    ...
  },
    "sql-file" : true
}
```

## Running Migrations

When first running the migrations, all will be executed in sequence. A table named `migrations` will also be created in your database to track which migrations have been applied.

      $ db-migrate up
      [INFO] Processed migration 20111219120000-add-pets
      [INFO] Processed migration 20111219120005-add-owners
      [INFO] Done

Subsequent attempts to run these migrations will result in the following output

      $ db-migrate up
      [INFO] No migrations to run
      [INFO] Done

If we were to create another migration using `db-migrate create`, and then execute migrations again, we would execute only those not previously executed:

      $ db-migrate up
      [INFO] Processed migration 20111220120210-add-kennels
      [INFO] Done

You can also run migrations incrementally by specifying a date substring. The example below will run all migrations created on or before December 19, 2011:

      $ db-migrate up 20111219
      [INFO] Processed migration 20111219120000-add-pets
      [INFO] Processed migration 20111219120005-add-owners
      [INFO] Done

You can also run a specific number of migrations with the -c option:

      $ db-migrate up -c 1
      [INFO] Processed migration 20111219120000-add-pets
      [INFO] Done

All of the down migrations work identically to the up migrations by substituting the word `down` for `up`.