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

Note:  In newer versions of db-migrate, we have included a promise-based interface.  In these newer versions, the `create` command will generate a file containing the following:

```javascript
exports.up = function(db) {
    return null;
};

exports.down = function(db) {
    return null;
};
```

All you have to do is populate these, invoking `callback()` or returning the result of your `db` operation when complete, and you are ready to migrate!

For example:

    $ db-migrate create add-pets
    $ db-migrate create add-owners

The first call creates `./migrations/20111219120000-add-pets.js`, which we can populate:

```javascript
/* Callback-based version */
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

```javascript
/* Promise-based version */
exports.up = function (db) {
  return db.createTable('pets', {
    id: { type: 'int', primaryKey: true },
    name: 'string'
  });
};

exports.down = function (db) {
  return db.dropTable('pets');
};
```

The second creates `./migrations/20111219120005-add-owners.js`, which we can populate:

```javascript
/* Callback-based version */
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

```javascript
/* Promise-based version */
exports.up = function (db) {
  return db.createTable('owners', {
    id: { type: 'int', primaryKey: true },
    name: 'string'
  });
};

exports.down = function (db) {
  return db.dropTable('owners');
};
```


Executing multiple statements against the database within a single migration requires a bit more care. You can either nest the migrations like:

```javascript
/* Callback-based version */
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

```javascript
/* Promise-based version */
exports.up = function (db) {
  return db.createTable('pets', {
    id: { type: 'int', primaryKey: true },
    name: 'string'
  })
  .then(
    function(result) {
      db.createTable('owners', {
        id: { type: 'int', primaryKey: true },
        name: 'string'
      });
    },
    function(err) {
      return;
    }
  );
};

exports.down = function (db) {
  return db.dropTable('pets')
    .then(
      function(result) {
        db.dropTable('owners');
      },
      function(err) {
        return;
      }
    );
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
var dbm;
var type;
var fs = require('fs');
var path = require('path');

/**
  * We receive the dbmigrate dependency from dbmigrate initially.
  * This enables us to not have to rely on NODE_PATH.
  */
exports.setup = function(options) {
  dbm = options.dbmigrate;
  type = dbm.datatype;
};

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

# Seeder Introduction

This page should give you an overview, of what seeders actually are and what you
can expect of them. To also tell some of the practices that make things easier
for you. This is really just an overview, take a look at the API description for
all details.

# General operations

The seeder consists of method such as update, insert and data manipulation
methods.

## update( table, data, searchclause/id )

While selecting data, that might not belong to your table yet, there are
different kinds of helpers, that are going to help you, while propagating your
database.

There are update helper and a defined where clause helper, included in functions
like select or update. An example would be this:

```javascript
exports.up = function( db ) {

  return db.update( 'test', { name: 'cedric', surname: 'camelot' }, 12 );
};
```

This is going to update the 12th record of the table test, which has got a new
name cedric camelot. There are more examples in our example project, which you
can find [here](). For an overview of all available options and helpers take a
look at the [API Description]().

# select( table, data, searchclause/id )

This helper is an option for you to use, if you want your seeders to be generic
and usable across different DBMS's. No matter if NoSQL or SQL or NewSQL, it
should behave similar in every case.

```javascript
exports.up = function( db ) {

  return db.select( 'test', [ 'id', 'name', 'surname' ], { bff: 12 } );
};
```

This example would result in the `id`, `name` and `surname` of the guys who have
number `12` as their bff.

# Common Helpers

These helpers are here for your when there are cases when you're working with
db-migrate and db-migrate could help you accomplishing your work more
efficiently and more easy.

## _l( options )

The lookup helper, a common task is that you have the value, but you do not have
necessarily the id of this value. This id might change for some reason, also it
should better never ever change but that by side, or you just really do not know
it. As far as this column is referencing a value in another table, this helper
is the one you are looking for.

## Options object

- t (shortcut: t) - DB to lookup the value
 - defaults only on compliance to db guidelines
- data column (shortcut: c) - Column with the corresponding value
 - defaults to 'value'
- id column (shortcut: k) - corresponding id column
 - defaults to 'id'

# DB-Migrate - DB Guidelines

There are some common practices that makes the work with DB-Migrate even easier.
These practices fit with already known best practices, that are not DB-Migrate
related. Also you can adjust many things to work with different standards.

# Column references

Some may always call them foreign keys... However, you can lookup without
actually providing a database.

If a column reference is referencing a value in the table test_db_arch, you
would name the column test_db_arch_id.

Beside this you can also do things like:
ref_test_db_arch_attribute
test_db_arch_attribute_fk

This would receive the value from the table test_db_arch and get the value from
the attribute column in this table. We always default the id column to be named
like `id`, there is no shorthand via the naming. Instead you can design a seed
like the following example:

```javascript
exports.up = function( db ) {

  return db.insert( 'test', {
    ref_test_db_arch_attribute: _l( { id: 'identifier' } ),
    test_db_arch_id: _l(),
    common_value: _l( { t: 'test_db_arch' } ),
    arch_id: _()
  } );
};
```

All of these, except `arch_id` are going to resolve to the value from the
`test_db_arch` table, referenced as id column. You can however also make
`arch_id` of this example work, you can register common shorthands. There are
many examples where you have columns of tables with long names, like
pseudo_type. Many do name their columns like `pt_id`, you can now register this
column via:

    db-migrate register alias pt pseudo_type

This makes you unable to use a table named `pt`, thus you really should know
what you're doing. Also to mention, it is recommended to have as less aliases
as possible, only use them if they're common and used all over your project and
by this means known to be used in this project.

## Change management

One of the most important things is to know when who is updating why what where,
if you do not known exactly what happens in parts of your project you're never
going to be able to do such things as zero downtime deployments or automation
of the deployment in general.
