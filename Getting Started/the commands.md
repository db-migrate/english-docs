# The commands

In the following, we will introduce you into the basic commands you need to use
db-migrate and unleash its power.

For a full list of the commands please view the command specification.


## up

The up command executes the migrations of your currently configured migrations
directory. More specific the up migrations are being called.

As default it does this for all migrations within the first *scope* of your
migrations directory. To limit the number of executed migrations you can
execute:

    db-migrate up -c 5

This will execute 5 migrations, but you can be also more specific. To execute
one migration by its name you can pass the name like the following:

    db-migrate up 20150207135259-myFancyMigration

## down

The down command executes the migrations of your currently configured
migrations directory. More specific the down migrations are being called.

Down migrations are called in reverse order in which the up migrations
previously were executed.

As default it does this for all migrations within the first *scope* of your
migrations directory. Also default is that down will only migrate the last
executed migration. To execute more then one down migration you can execute:

    db-migrate down -c 5

If you want to let *db-migrate* execute all down migrations you can also call:

    db-migrate reset

## reset

The reset command is a shortcut to execute all down migrations and literally
reset all migrations which where currently done. The reset command also
executes by default only the first *scope*

## create

The create command creates templates for you, there are several options for
this. This is useful if you start a new migration, thus you get a proper
formatted filename and a boilerplate. Thanks to this, you can keep
concentrating on the important part: creating a migration.

For example this commands:

    db-migrate create migrationname

Creates a default migration with the name migrationname in your configured
migrations directory.

    db-migrate create anothermigration --coffee-file

Creates a coffee script migration with the name anothermigration in your
configured migrations directory.

    db-migrate create anothermigration --sql-file

Creates a migration that loads sql file with the name anothermigration in your
configured migrations directory. This is useful if your transitioning from not
using migrations. Another way to get started with migrations, with an already
existing database is to use another project from wzrdtales:

[umigrate](https://github.com/wzrdtales/node-ultimate-migrate)

Umigrate is a tool that can generate migrations from your schema and is also
able to generate only the differences of two databases. Currently it only
supports MariaDB and thus also mysql.

# db

## db:create

The db:create command creates a database using your current configuration.

    db-migrate db:create testDB

This command would create a database named testDB.

## db:drop

The db:drop command drops a database using your current configuration.

    db-migrate db:drop testDB

This command would drop a database named testDB.

# Scoping

Additional you can use scoping in db-migrate.

So what does this mean?

If you enter any command from above with a doublepoint and the name of your
scope, like the following:

    db-migrate up:test

Then db-migrate will only execute all migrations in the "test" scope. So where
exactly are the scopes.

We assume you have configured your migrations directory to the directory
"migrations". In this case the scope "test" has the following path:

migrations/test/

If you don't enter a scope, by default only the migrations within the
migrations directory itself will be executed. Now that we have entered the
scope "test" all migrations within `migrations/test` will be executed.

The scopes can be used with all basic commands, such as:

    db-migrate down:test
    db-migrate reset:test
    db-migrate create:test

If you want to execute all existent scopes, you can execute the following
command:

    db-migrate up:all

`all` is a keyword in db-migrate and can thus not be used as scope name.
Instead the all scope executes all scopes that exist in your migrations
directory.

Another feature of scopes are multiple configurations. See the point below.

### Scope Configuration

You can also configure the scope to specify a sub configuration. Currently you can only define database and schema within this config.

This config file is used to tell db-migrate to switch to the `database` or
`schema`. Databases is used for most databases, except **postgres**
which needs the schema variable.

It's currently also not possible to switch the database over this config with **postgres**.

```json
{
  "database": "test",
  "schema": "test"
}
```

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
exports.up = function( up ) {

  return db.select( 'test', [ 'id', 'name', 'surname' ], { bff: 12 } );
};
```

This example would result in the `id`, `name` and `surname` of the guys who have
number `12` as their bff.

# Common Helpers

These helpers are here for your when there are cases when you're working with
db-migrate and db-migrate could help you accomplishing your work more efficiently and more easy.

## _l( options )

The lookup helper, a common task is that you have the value, but you do not have
necessarily the id of this value. This id might change for some reason, also it should better never ever change but that by side, or you just really do not know it. As far as this column is referencing a value in another table, this helper is the one you are looking for.

## Options object

- t (shortcut: t) - DB to lookup the value
 - defaults only on compliance to db guidelines
- data column (shortcut: c) - Column with the corresponding value
 - defaults to 'value'
- id column (shortcut: k) - corresponding id column
 - defaults to 'id'
