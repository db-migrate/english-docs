## Installation

## New Instructions Since (v0.10.x)

To use db-migrate you need to install it globally first:

    $ npm install -g db-migrate

If you want to use db-migrate as local module now you can install it in your
local modules:

    $ npm install db-migrate

The following command will execute **always** your local version of db-migrate
if you have installed a local version. If it does not find any local version
in your current directory it executes the globally installed version.

To use db-migrate you can now use:

    $ db-migrate

### Using db-migrate in tests

You want to use db-migrate for example with travis ci? You need to do one of
the following things:

 * Install it via package.json and call it via
    $ node node_modules/db-migrate/bin/db-migrate

 * Install it globally via .travis.yml config and call it via
    $ db-migrate

## Basic Usage

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


## Old Installation instructions

    $ npm install -g db-migrate

DB-Migrate is now available to you via:

    $ db-migrate

### As local module

Want to use db-migrate as local module?

    $ npm install db-migrate

DB-Migrate is now available to you via:

    $ node node_modules/db-migrate/bin/db-migrate
