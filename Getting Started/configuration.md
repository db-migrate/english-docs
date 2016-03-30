## Configuration

db-migrate supports the concept of environments. For example, you might have a dev, test, and prod environment where you need to run the migrations at different times. Environment settings are loaded from a database.json file like the one shown below:

```javascript
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
```javascript
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

```javascript
{
  "defaultEnv": "local",
  "local": {
    "driver": "sqlite3",
    "filename": ":memory:"
  }
}
```

In addition, the default env can also be set with an environment variable. This can be helpful if you'd like to use the `NODE_ENV` variable to select configuration:

```javascript
{
  "defaultEnv": {"ENV": "NODE_ENV"},
  "prod": {
    "driver": "mysql",
    "user": {"ENV": "PRODUCTION_USERNAME"},
    "password": {"ENV": "PRODUCTION_PASSWORD"}
  },
}
```

Alternatively, you can specify a DATABASE_URL
environment variable that will be used in place of the configuration
file settings. This is helpful for use with Heroku.



## Important - For MySQL users

If you use MySQL, to be able to use multiple statements in your sql file, you have to set the property `multiple-statements: true` when creating the connection object. You can set it in your `database.json` as follows:

```
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
