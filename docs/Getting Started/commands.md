# Commands

    db-migrate <command>[:scope] [argument] [options]

Every command reads the [configuration](configuration.md) of the current
environment and exits with code 1 if it fails. The [options](#options) are
listed at the end of this page.

## up

Runs the migrations that did not run yet, in the order of their timestamps.

    db-migrate up

Limit the number of migrations with `--count`:

    db-migrate up -c 5

Or run all pending migrations up to and including a given one. The name only
needs to be a prefix, so a date works as well:

    db-migrate up 20150207135259-myFancyMigration
    db-migrate up 20150207

Without a scope, `up` runs the migrations directly in the migrations
directory, see [Scoping](#scoping) for sub folders.

## down

Reverts executed migrations, in the reverse order of their execution. By
default only the last one:

    db-migrate down

Revert more with `--count`:

    db-migrate down -c 5

Or revert every migration executed after a given one, the given migration
itself stays:

    db-migrate down 20150207135259-myFancyMigration

## reset

Reverts all executed migrations of the scope.

    db-migrate reset

## sync

Migrates up or down to a given migration, db-migrate picks the direction. The
destination is required.

    db-migrate sync 20150207135259-myFancyMigration

If the latest executed migration is newer than the destination, everything
executed after the destination is reverted. Otherwise the pending migrations
up to and including the destination are run.

## check

Lists the pending migrations without running them.

    $ db-migrate check
    [INFO] Files to run: [ '20150207135259-myFancyMigration' ]

The option `--check` does the same for `up`, `down`, `reset` and `fix`: it
prints which migrations they would run, with count and destination applied.

    db-migrate down -c 3 --check

## fix

Rebuilds the schema db-migrate learned from the executed
[v2 migrations](../Guides/migrations v2.md). It replays them into the state
table only, nothing is executed on the database. v1 migrations are skipped,
they do not teach db-migrate a schema.

    db-migrate fix

With `--backup-state` the stored schema is backed up first, into a file
`<state-table>_b_<timestamp>.dbmigrate` in the current directory, and the
state table is renamed to `<state-table>_b_<timestamp>` before a new one is
created.

    db-migrate fix --backup-state

Before db-migrate 1.6.0, `fix` learned on top of the stored schema and added
the records of the migrations a second time, so `down` reverted their steps
twice afterwards. If you ran `fix` with an older version, run it again with
1.6.0.

## create

Creates a new migration from a template. The file name is the current UTC
time followed by the given name.

    db-migrate create add-pets

creates `migrations/20150207135259-add-pets.js`, a v1 migration with an `up`
and a `down` function, see [Usage](usage.md).

| Option | Creates |
|---|---|
| none | a v1 migration |
| `--v2-file` | a [v2 migration](../Guides/migrations v2.md) |
| `--sql-file` | a v1 migration running `sqls/<name>-up.sql` and `sqls/<name>-down.sql`, which are created as well |
| `--sql-file --ignore-on-init` | like `--sql-file`, but the up migration is skipped when running with `--ignore-on-init` |
| `--coffee-file` | a CoffeeScript migration (`.coffee`) |
| `--coffee-file --sql-file` | a CoffeeScript migration running sql files |
| `--sql` | a plain SQL migration, needs [db-migrate-plugin-sql](plugins.md#plain-sql-migrations) |

The template options can also be set permanently, as a top level key of the
`database.json` or in the [rc config](configuration.md#rc-configs):

```json
{
  "dev": { "driver": "sqlite3", "filename": "dev.db" },
  "sql-file": true
}
```

db-migrate itself only loads `.js` migrations. To run CoffeeScript migrations
you need a [plugin](../Developers/writing plugins.md) adding the `coffee`
extension.

A name containing slashes creates the migration in sub folders:

    db-migrate create users/add-email

creates `migrations/users/20150207135259-add-email.js`. With a scope,
`db-migrate create:users add-email` creates it in the folder of the scope
`users` as well.

`create` also writes a `package.json` with `"type": "commonjs"` into the
folder of the migration, so migrations are loaded as CommonJS even if your
project uses ES modules.

## db:create and db:drop

Create or drop a database, with the connection settings of the current
environment, without its `database`.

    db-migrate db:create testDB
    db-migrate db:drop testDB

db-migrate asks the driver to create the database only if it does not exist
and to drop it only if it exists. See the [driver pages](../drivers.md) for
what each driver supports. `db` without `:create` or `:drop` fails with
`Missing db command, use db:create or db:drop`, a missing name with
`You must enter a database name!`.

## seed

    db-migrate seed [name]
    db-migrate seed down [name]
    db-migrate seed reset

Runs the [seeds](../Guides/seeds.md) in `seeds/`, all or the one named,
replacing the rows they inserted before. `seed down` removes the rows of all
seeds or the one named, `seed reset` of all seeds. Set the directory with
`--seeds-dir`. Since db-migrate 1.3.0, before it failed with `Seeders are not
supported by db-migrate 1.0`.

The options `--seeds-table`, `--vcseeder-dir` and `--staticseeder-dir` of the
seeders of 0.11 are still accepted, but have no effect.

## work

    db-migrate work [--parallel n] [--pause ms] [--batch n] [--watch]

Runs the jobs of [background migrations](../Guides/background migrations.md),
until none is left or, with `--watch`, until stopped with `SIGINT` or
`SIGTERM`. Since db-migrate 1.6.0.

## Commands of plugins

Plugins can add commands, see [Plugins](plugins.md). An unknown command fails
with `Invalid Action` and prints the help.

# Scoping

Scopes are sub folders of the migrations directory, each with its own list of
migrations. Add the scope to the command, separated by a colon:

    db-migrate up:test

runs the migrations in `migrations/test/`. Without a scope, only the
migrations directly in `migrations/` are run, sub folders are left alone.
Scopes work with `up`, `down`, `reset`, `sync`, `check`, `fix` and `create`:

    db-migrate down:test
    db-migrate reset:test
    db-migrate create:test add-pets

Scopes can be nested, `db-migrate up:test/unit` runs `migrations/test/unit/`.

The migrations table records the migrations of a scope with the scope as
prefix, like `test/20150207135259-add-pets`, so equally named migrations of
different scopes do not collide.

## The scope all

`all` is reserved and can not be used as a scope name. With `all`, a command
runs for every scope, one after another: first the migrations directly in the
migrations directory, then every sub folder, recursively and in alphabetical
order, nested ones as `a/b`. Folders named `sqls` hold the files of
`--sql-file` migrations and are no scopes.

    db-migrate up:all
    db-migrate check:all

This works with `up`, `down`, `reset`, `sync`, `check` and `fix`. With
`--force-exit`, db-migrate exits after the last scope only.

## Scope configuration

A scope can have a configuration of its own, a `config.json` in the folder of
the scope:

    migrations/test/config.json

Entries in the `{"ENV": "NAME"}` notation are read from the environment, like
in the [config file](configuration.md#environment-variables). The
configuration only applies to the scope itself, not to scopes nested in it.

### Switching the database or schema

A `config.json` with only `database` or `schema` switches the connections of
the environment before the migrations of the scope run:

```json
{
  "database": "test"
}
```

How depends on the driver: mysql switches the database, PostgreSQL sets the
`search_path` to the given `database` or `schema` instead, as a connection
can not switch databases there, and sqlite3 ignores it. See the
[driver pages](../drivers.md).

The migrations table and the state table are used in the database or schema
switched to, so the scope keeps its own migration records, lock, recovery
progress and learned schema there.

**Upgrading from 1.0:** db-migrate 1.0 kept the state of such scopes in the
database of the environment. If v2 migrations of the scope ran before, run
`db-migrate fix:<scope>` once to learn their schema in the database of the
scope.

### A database of its own

A `config.json` with any other connection setting, like `host`, `user`,
`password` or even `driver`, connects the scope on its own. It inherits every
setting of the environment it does not set itself:

```json
{
  "host": "analytics.internal",
  "database": "analytics",
  "user": "migrator",
  "password": {"ENV": "ANALYTICS_PASSWORD"}
}
```

The scope has its own migrations table, state table, lock and learned schema
in that database. With PostgreSQL, `database` means the real database here,
not the `search_path`.

# Options

Options are given on the command line, or in an
[rc config](configuration.md#rc-configs) under the same names.

| Option | Default | Description |
|---|---|---|
| `--env`, `-e` | `dev`, then `development` | The environment of the config file to use, see [Configuration](configuration.md). |
| `--config` | `./database.json` | The config file. |
| `--migrations-dir`, `-m` | `./migrations` | The directory of the migrations. |
| `--count`, `-c` | all for `up`, 1 for `down` | Maximum number of migrations to run. |
| `--dry-run` | `false` | Print the SQL instead of running it, see below. |
| `--check` | `false` | Print the migrations to run without running them. |
| `--verbose`, `-v` | `false` | Verbose output, including the settings used (with the password masked) and the complete error of the driver. |
| `--log-level` | all | The log levels to print, out of `sql`, `info`, `warn` and `error`, separated by `\|`, e.g. `--log-level "warn\|error"`. |
| `--table`, `--migration-table`, `-t` | `migrations` | Name of the table recording the executed migrations. |
| `--state-table`, `--state`, `-s` | `migrations_state` | Name of the [state table](configuration.md#state-table-and-migration-lock). |
| `--lock-timeout` | `60000` | Milliseconds without any sign of life after which the [migration lock](../Guides/running in parallel.md) of another process is taken over. |
| `--lock-interval` | `1000` | Milliseconds between checks while waiting for the migration lock. |
| `--non-transactional` | `false` | Do not run v1 migrations inside a transaction. |
| `--force-exit` | `false` | Exit the process with `process.exit(0)` after a successful run. |
| `--ignore-completed-migrations` | `false` | Ignore the record of executed migrations and start at the first migration, all migrations run again. |
| `--v2-file` | `false` | `create`: create a v2 migration. |
| `--sql-file` | `false` | `create`: create a migration running sql files. |
| `--coffee-file` | `false` | `create`: create a CoffeeScript migration. |
| `--ignore-on-init` | `false` | `create` with `--sql-file`: create a migration whose up is skipped when running with `--ignore-on-init`, e.g. to initialize a database from a dump. `up`: skip the up of such migrations, they are recorded as executed nevertheless. |
| `--backup-state` | `false` | `fix`: back up the state first. |
| `--help`, `-h` | | Print the help. |
| `--version`, `-i` | | Print the version. |

## Dry run

`--dry-run` prints the SQL of the migrations, including the statements
db-migrate runs on its own, instead of executing it. Nothing is recorded and
the migration lock is not taken. With `--log-level sql` only the SQL is
printed, with the parameters inlined:

    db-migrate up --dry-run --log-level sql

Queries run with `all` in a migration are executed even in a dry run, see the
[SQL API](../API/SQL.md#allsql-params-callback).
