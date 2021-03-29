## Configuration

db-migrate supports the concept of environments. For example, you might have a dev, test, and prod environment where you need to run the migrations at different times. Environment settings are loaded from a `database.json` file like the one shown below:

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
    "port": "20144",
    "ssl": "true",
    "schema": "my_schema"
  },

  "mongo": {
    "driver": "mongodb",
    "database": "my_db",
    "host": "localhost"
  },

  "other": "postgres://uname:pw@server.com/dbname"
}
```

You can also specify environment variables in your config file by using a special notation. Here is an example:
```json
{
  "prod": {
    "driver": "mysql",
    "user": {"ENV": "PRODUCTION_USERNAME"},
    "password": {"ENV": "PRODUCTION_PASSWORD"}
  },
}
```
In this case, db-migrate will search your environment for variables
called `PRODUCTION_USERNAME` and `PRODUCTION_PASSWORD`, and use those values for the corresponding configuration entry.

If you use the [dotenv](https://www.npmjs.com/package/dotenv) package to manage environment variables, db-migrate will automatically load it.

Note that if the settings for an environment are represented by a single string that string will be parsed as a database URL.

You can pass the -e or --env option to db-migrate to select the environment you want to run migrations against. The --config option can be used to specify the path to your database.json file if it's not in the current working directory.

    db-migrate up --config config/database.json -e prod

The above will run all migrations that haven't yet been run in the prod environment, grabbing the settings from config/database.json.

If the environment is not specified by the -e or --env option, db-migrate will look for an environment named `dev` or `development`. You can change this default behavior with the database.json file:

```json
{
  "defaultEnv": "local",
  "local": {
    "driver": "sqlite3",
    "filename": ":memory:"
  }
}
```

In addition, the default env can also be set with an environment variable. This can be helpful if you'd like to use the `NODE_ENV` variable to select configuration:

```json
{
  "defaultEnv": {"ENV": "NODE_ENV"},
  "prod": {
    "driver": "mysql",
    "user": {"ENV": "PRODUCTION_USERNAME"},
    "password": {"ENV": "PRODUCTION_PASSWORD"}
  },
}
```

## DATABASE_URL

Alternatively, you can specify a `DATABASE_URL` environment variable that will
be used in place of the configuration file settings. This is helpful for use
with Heroku.

**Note**: If a database url is specified, the config file is being skipped. You
can however also specify rc configs, where you can configure everything you can
configure also on the CLI.

## RC configs

RC configs give the possibility to configure settings for more than just one
project, as RC configs are being loaded from different directories.

You can take a view over [here](https://github.com/dominictarr/rc#standards)
where to save those configs.

Most prominent locations are, the root directory where you currently execute
db-migrate and your `HOME` directory. The file is always named `.db-migraterc`,
except for some examples you can find under the link above.

An example `.db-migraterc` config file could look like this:

```json
{
  "sql-file": true,
  "configFile": "path/to/config/database.json",
  "migrations-dir": "path/to/migrations",
  "table": "new_migration_table_name"
}
```
Use `table` property in `.db-migraterc` config file to change the default name of migrations table.

This would set activate the sql mode unless you would deactivate it in your
database.json again, which always has the highest priority.

The `configFile` is a special rc config variable, because `config` is reserved by the `rc` module.

## Important - For MySQL users

If you use MySQL, to be able to use multiple statements in your sql file, you have to set the property `multipleStatements: true` when creating the connection object. You can set it in your `database.json` as follows:

```json
{
  "dev": {
    "host": "localhost",
    "user": { "ENV" : "DB_USER" },
    "password" : { "ENV" : "DB_PASS" },
    "database": "database-name",
    "driver": "mysql",
    "multipleStatements": true
  }
}
```
