# Configuration

## Environments

db-migrate supports the concept of environments. For example, you might have
a dev, test, and prod environment where you need to run the migrations at
different times. The environments are read from a `database.json` in the
current directory:

```json
{
  "dev": {
    "driver": "sqlite3",
    "filename": "~/dev.db"
  },

  "test": {
    "driver": "sqlite3",
    "filename": ":memory:"
  },

  "prod": {
    "driver": "mysql",
    "user": "root",
    "password": "root"
  },

  "pg": {
    "driver": "pg",
    "user": "test",
    "password": "test",
    "host": "localhost",
    "database": "mydb",
    "port": "20144"
  },

  "mongo": {
    "driver": "mongodb",
    "database": "my_db",
    "host": "localhost"
  },

  "other": "postgres://uname:pw@server.com/dbname"
}
```

Every environment names its `driver`: `pg` (also `postgres` and
`postgresql`), `mysql`, `sqlite3` (also `sqlite`), `cockroachdb`, `mongodb`,
or the name of any other driver installed as `db-migrate-<name>`. A driver
can also be loaded from a path or module name:

```json
{
  "dev": {
    "driver": { "require": "./my-driver" }
  }
}
```

All other settings are passed on to the driver, see the
[driver pages](../drivers.md) for the settings of each driver. `username` is
accepted as an alias of `user`.

Select the environment with `-e` or `--env`, and the config file with
`--config` if it is not `./database.json`:

    db-migrate up --config config/database.json -e prod

## The default environment

Without `--env`, db-migrate uses the environment `dev`, or `development` if
there is no `dev`. Change this with `defaultEnv` (or its alias `default`):

```json
{
  "defaultEnv": "local",
  "local": {
    "driver": "sqlite3",
    "filename": ":memory:"
  }
}
```

The default environment can also come from an environment variable, for
example `NODE_ENV`:

```json
{
  "defaultEnv": {"ENV": "NODE_ENV"},
  "prod": {
    "driver": "mysql",
    "user": {"ENV": "PRODUCTION_USERNAME"},
    "password": {"ENV": "PRODUCTION_PASSWORD"}
  }
}
```

## Environment variables

Any setting can be read from an environment variable with the notation
`{"ENV": "NAME"}`, at any depth:

```json
{
  "prod": {
    "driver": "mysql",
    "user": {"ENV": "PRODUCTION_USERNAME"},
    "password": {"ENV": "PRODUCTION_PASSWORD"}
  }
}
```

db-migrate replaces them with the values of `PRODUCTION_USERNAME` and
`PRODUCTION_PASSWORD`. An empty variable is reported with `--verbose`.

A whole environment can be read from a variable holding a database URL:

```json
{
  "prod": {"ENV": "DATABASE_URL_PROD"}
}
```

db-migrate loads a `.env` file from the current directory with
[dotenv](https://www.npmjs.com/package/dotenv) before reading the config. Set
`dotenvCustomPath` in the [rc config](#rc-configs) to load another file.

## Database URLs

An environment given as a string is parsed as a database URL:

```json
{
  "prod": "postgres://user:password@db.example.com:5432/app?ssl=true"
}
```

The scheme becomes the driver, query parameters become settings, like `ssl`
above. sqlite3 takes the file name as path, `sqlite3:///var/app.db` or
`sqlite3:app.db`.

An environment can also combine a `url` with further settings:

```json
{
  "prod": {
    "url": "postgres://user:password@db.example.com/app",
    "schema": "app"
  }
}
```

### DATABASE_URL

If the environment variable `DATABASE_URL` is set, its settings are applied to
every environment of the config file that is an object, overriding the same
settings there. Without a config file, db-migrate connects to `DATABASE_URL`
alone. This is helpful with hosting providers like Heroku.

## overwrite and addIfNotExists

An environment can overwrite settings, for example the ones coming from a
`url` or `DATABASE_URL`, or add settings only if they are not set:

```json
{
  "prod": {
    "url": {"ENV": "DATABASE_URL"},
    "overwrite": {
      "database": "app"
    },
    "addIfNotExists": {
      "port": 5432
    }
  }
}
```

## Settings outside of the environments

Top level keys of the config file which are no environment set defaults for
`create`, e.g. `"sql-file": true` creates every migration with sql files, see
[Commands](commands.md#create).

## RC configs

RC configs set the [options](commands.md#options) of db-migrate for more than
one project, or just to avoid typing them. They are read by
[rc](https://github.com/dominictarr/rc#standards) from these files, later ones
taking precedence:

1. `/etc/db-migrate/config`, `/etc/db-migraterc`
2. `~/.config/db-migrate/config`, `~/.config/db-migrate`,
   `~/.db-migrate/config`, `~/.db-migraterc`
3. `.db-migraterc` in the current directory, or the first one found in a
   parent directory

The keys are the long names of the options, except for the table names: use
`table` for the migrations table and `state` for the state table, the names
`migration-table` and `state-table` are not picked up from rc configs. Options
given on the command line take precedence over the rc configs.

```json
{
  "configFile": "path/to/config/database.json",
  "table": "new_migration_table_name",
  "migrations-dir": "db/migrations",
  "sql-file": true,
  "lock-timeout": 120000,
  "dotenvCustomPath": "config/.env"
}
```

`configFile` sets the path of the config file, `config` is reserved by rc.

## SSH tunnels

To connect through an ssh tunnel, install
[db-migrate-plugin-tunnel-ssh](plugins.md#ssh-tunnels) and add a `tunnel`
section to your environment.

## State table and migration lock

db-migrate keeps a state table next to the migrations table, `migrations_state`
by default, set with `--state-table` or `state` in the rc config. It holds
the [migration lock](../Guides/running in parallel.md) and the schema and
progress of [v2 migrations](../Guides/migrations v2.md). The lock is tuned
with `lock-timeout` and `lock-interval`. Keep the table, deleting it loses the
lock and the information needed to revert v2 migrations and to recover
interrupted runs.
