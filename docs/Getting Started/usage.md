# Usage of db-migrate

db-migrate is used from the command line:

    db-migrate [up|down|check|reset|sync|create|db] [[dbname/]migrationName|all] [options]

See [Commands](commands.md) for every command and option. db-migrate can also
be used as a module, see the [programmable API](../API/programable.md).

## Creating Migrations

To create a migration, execute `db-migrate create` with a title. db-migrate
creates a node module within `./migrations/`:

```javascript
'use strict';

var dbm;
var type;
var seed;

/**
  * We receive the dbmigrate dependency from dbmigrate initially.
  * This enables us to not have to rely on NODE_PATH.
  */
exports.setup = function(options, seedLink) {
  dbm = options.dbmigrate;
  type = dbm.dataType;
  seed = seedLink;
};

exports.up = function(db) {
  return null;
};

exports.down = function(db) {
  return null;
};

exports._meta = {
  "version": 1
};
```

This is a v1 migration: `up` migrates, `down` reverts it. For migrations
without a down function, see [Migration schema v2](../Guides/migrations v2.md).

`db` is the driver of your database, see the [SQL API](../API/SQL.md) and the
[NoSQL API](../API/NoSQL.md) for what it offers. Every operation returns a
promise and accepts a callback as last argument as well. A migration either
returns a promise, or takes a callback as last argument and calls it when it
is done.

`setup` is optional. It is called before `up` or `down` with these options:

| Option | |
|---|---|
| `dbmigrate` | `{ version, dataType }`, the version of db-migrate and the [data types](../API/generic datatypes.md) |
| `type` | the data types |
| `dryRun` | `true` when running with `--dry-run` |
| `cwd` | the working directory |
| `noTransactions` | `true` when running with `--non-transactional` |
| `verbose` | `true` when running with `--verbose` |
| `ignoreOnInit` | `true` when running with `--ignore-on-init` |
| `log` | the logger of db-migrate, with `info`, `warn`, `error` and `verbose` |
| `Promise` | the promise library of db-migrate (bluebird) |

`seedLink` is a remnant of the seeders and is always undefined in 1.0.

For example:

    $ db-migrate create add-pets
    $ db-migrate create add-owners

The first call creates `./migrations/20111219120000-add-pets.js`, which we can
populate:

```javascript
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

The same with a callback:

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

Several operations in one migration are easiest with an async function:

```javascript
exports.up = async function (db) {
  await db.createTable('owners', {
    id: { type: 'int', primaryKey: true },
    name: 'string'
  });
  await db.addColumn('pets', 'owner_id', { type: 'int' });
};

exports.down = async function (db) {
  await db.removeColumn('pets', 'owner_id');
  await db.dropTable('owners');
};
```

With callbacks, nest them:

```javascript
exports.up = function (db, callback) {
  db.createTable('owners', {
    id: { type: 'int', primaryKey: true },
    name: 'string'
  }, function (err) {
    if (err) return callback(err);
    db.addColumn('pets', 'owner_id', { type: 'int' }, callback);
  });
};
```

### Transactions

v1 migrations run inside a transaction of the driver (PostgreSQL, CockroachDB,
MySQL, sqlite3), together with writing their record into the migrations table.
Whether a failed migration is rolled back completely depends on whether your
database can roll back schema changes. Disable the transaction with
`--non-transactional`. db-migrate does not revert a failed v1 migration on its
own, see [Failures and recovery](../Guides/failures and recovery.md).

### Using files for sqls

If you prefer to write your up and down statements in sql files, use the
`--sql-file` option, it creates the files and the javascript code loading
them.

To write migrations as plain SQL files, without any JavaScript, use the
[db-migrate-plugin-sql](plugins.md#plain-sql-migrations) plugin instead.

    $ db-migrate create add-people --sql-file

This creates 3 files:

```
./migrations/20111219120000-add-people.js
./migrations/sqls/20111219120000-add-people-up.sql
./migrations/sqls/20111219120000-add-people-down.sql
```

The sql files contain:

```sql
/* Replace with your SQL commands */
```

and the javascript file reads them and runs their content with `runSql`:

```javascript
exports.up = function(db) {
  var filePath = path.join(__dirname, 'sqls', '20111219120000-add-people-up.sql');
  return new Promise( function( resolve, reject ) {
    fs.readFile(filePath, {encoding: 'utf-8'}, function(err,data){
      if (err) return reject(err);
      console.log('received data: ' + data);

      resolve(data);
    });
  })
  .then(function(data) {
    return db.runSql(data);
  });
};
```

Whether a file may contain several statements depends on the driver, for
MySQL enable [multipleStatements](../Drivers/mysql.md).

To always create sql files, set `"sql-file": true` as a top level key of the
`database.json` or in the [rc config](configuration.md#rc-configs).

With `--sql-file --ignore-on-init`, the up migration only runs the up sql file
if db-migrate is not run with `--ignore-on-init`. This is useful for
migrations creating what a fresh database got already from a dump.

## Running Migrations

When first running the migrations, all of them are executed in sequence. The
table `migrations` is created to track which migrations have been applied.

      $ db-migrate up
      [INFO] [migration] Processed 20111219120000-add-pets
      [INFO] [migration] Processed 20111219120005-add-owners
      [INFO] Done

Subsequent runs only execute what is new:

      $ db-migrate up
      [INFO] [migration] Nothing to run
      [INFO] Done

Run migrations up to a date by giving a prefix of their name. The example runs
all migrations created on or before December 19, 2011:

      $ db-migrate up 20111219

Or a specific number of migrations with `-c`:

      $ db-migrate up -c 1

`db-migrate down` works the same way in the other direction, by default it
reverts the last migration. See [Commands](commands.md).
