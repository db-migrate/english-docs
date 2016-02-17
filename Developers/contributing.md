# Contributing

So happy to see you here. Any help is appreciated and so is yours!

So let us get you started!

## Guidelines

Before reading about the technical stuff, it's important to know to which
guidelines the project sticks.

## The workflow

We advise you to follow the following steps, if you plan to contribute:

1. Choose the right project you want to contribute to, either a driver or
db-migrate itself.

2. Fork the repo.

3. Create a new branch to work with out of the master branch.

4. Setup your environment and let the current tests run, they should pass
if not there is something wrong with your setup.

5. Be awesome!

6. Add a test for your change. Refactoring and documentation changes require no new tests. If you are adding functionality or fixing a bug, please include a test.

7. Make the test pass.

8. Pull the remote master into your master branch and rebase
your current working branch against this master.

9. Push your rebased branch and submit your pull request.

10. Make sure the travis tests in your pull request are all passing, if they
fail check why.

## Tips

 * You should never work in your master branch of your fork, you're going to
pull the changes from the master project into this branch and use your
branch to rebase against it.

 * You should pull changes from the master project regularly and rebase against
them to make your own life easier!

## The Developer getting started

Just as the user, we want you to find an easy start into the code. If you
follow the following guide, most of your question should be resolved before
they arise.

### Splitted Projects

To enable the best possible flexibility, db-migrate is from v0.10.x designed
to be extendable. This is the reason why the drivers aren't anymore in the
db-migrate project. Instead you will find them in their own.

This also means everyone can create anytime a new driver, without directly
contributing to the project. If you want that your driver is going to be
maintained by us you have obviously to work with us and the repository to your
driver have to enter the db-migrate organization.

Ok enough introduction now, let's get you started, click one of the following
links if you want to:

  * [Create your own driver](#creating-your-own-driver)

  * [Contribute to db-migrate itself](#contributing-to-db-migrate)


## Contributing to db-migrate

DB-Migrate consists out of a few parts. The migrator, seeder, drivers, usage 
API and programable API.
If you plan to make changes to the programable or usage API, always keep in
mind that backwards compatibility must be ensured.

This means, the behavior of functions may not be modified! It is possible to
extend them to provide additional functionality, but not to change the behavior
and thus destroying functionality.
If you realize, you need this new behavior, create a new function and functions
get deprecated if your new functionality is going to replace it in one of the
next major releases.

## Functionality description of db-migrate

DB-Migrate is a database toolset which provides the ability to alter the schema
of any database system of which a db-migrate driver exists. This includes
creating and deleting tables and databases as all standard operations, such as
changing an existing column, deleting and them and so on.

While providing this abilities, DB-Migrate does strictly only provide the 
ability to represent a state of the database at a given time. It is not a
full featured database management tool and wont provide the ability to execute
operations which are not related to the concept of version controlling changes
to the schema of a database. This excludes for example the ability to create a
dump of the database or the schema. 
To note: If you want to archive the latter one you might take a look at 
[umigrate](https://travis-ci.org/wzrdtales/node-ultimate-migrate) which is the 
partner project of db-migrate which adds the ability to generate migrations out
of a given schema and also provides the ability to generate migrations for just
the differences between existing migrations and the current schema.

DB-Migrate is strictly requires to support standard data definition operations.
This also applies to NoSQL Schemas, where for example in the case of mongodb
a cóllection is treated as the equivalent to a table.

DB-Migrate differenciate between migrations and seeding operations and strictly
provides only the interface which should be available to those. Therefore there
may not added new functionality to those, that does not fit their description.
An interface might only contain functionality does not fit this restriction, 
because of backward compatibility and must be deprecated and marked to be 
removed on the next major release.

The descriptions of those are defined as of the following.

### Description of the migrator interface
The migrator interface provides the ability to handle all abilities which are 
needed to successfully modify the DD of your tables.
This includes all DDL methods provided by SQL naturally.

### Description of the seeder interface

The seeder interface provides the ability to handle all operations, that
are not DDL specific and thus not a migration.

These operations are currently, but not limited to:

 * inserting data
 * removing data
 * searching data
 * truncating whole tables

This functionality is provided in two ways to the user. First there are
traditional seeder. You can call them whenever you want, and how often
you want. This results into a use case, that make seeders might only used
for development and not to use them in a deployment process.

Here the second way of usage steps in, the version controlled seeders, in
short VC Seeds.
There is technically no big difference between them, except the following
details:

A VC Seed can be called from a migration, via seed.execute and seed.link. A
normal seeder can not. Also A VC Seeder has a down and up function, like
the way the migrations work, the static instead has a truncate function,
which gets called before the seed function. As a seeder often wants to
truncate tables or just delete data from a table.
And last but not least, a VC Seed can not be executed if it was executed
before, you need to roll back it first, it's in short handled like
migrations are. A normal seed can be executed just as often as you want
without rollback your data at any time.

To note: If you rollback a migration, linked to a seeder, db-migrate will
also rollback the seed. This is also a reason why you can't rollback a
specific migration, you would going to break that much, you probably loose
a bunch of valueable time.

### Start developing on db-migrate

At first take a look at the source, there should be an available documentation
for the most of the functions. If not you can naturally ask in our chat how 
things work, this is also the way you ensure that this component gets 
documented afterwards.
We currently do not create a separate documentation out of the code docs.

### Promisify everything

The functionality description of db-migrate describes that every function of
db-migrate which is async should be a promise, but if part of a driver, because
of backwards compatibility, also should accept a callback function as last 
parameter.

The used promise library must be bluebird.

## Creating your own driver

While you create your own driver, you've got the freedom to do it without 
binding to any existing codestyles and conventions. But keep in mind, you're
responsible for your driver and if you want it to be officially supported by
db-migrate, you still need to all conventions like you would do on a normal
contribution.

Ok, so let's start with the actual details!

Before we start, make sure you have initialized a new node js project in your
current folder. If not do this via:

    $ npm init

The first thing we do, is to install the base for our driver. You do this
simply by executing the following command:

    $ npm install db-migrate-base --save

Ok, now two more dependency you should always install. Bluebird, for the
promises. Without promises your driver probably doesn't find acceptance and
would never be treated as official driver, by the db-migrate team. And also
don't forget moment, you probably need it for our dates.

    $ npm install bluebird --save 
    $ npm install moment --save

#### Ctor and co.

Great! Now it's time to create our `index.js`. We extend the base we installed
before and overwrite or use its functionality. Take a look at the 
[base driver](https://github.com/db-migrate/db-migrate-base/blob/master/index.js)
to get an overview, what is actually available from the beginning. So we have
our extended Base class, next we need to export a connect method in which we
initialize our driver and return an instance of it and also we get informations
from db-migrate, such as version information and functionality we can use.
We may also want to create constructor, this is done by defining a function in
our extended base which is named init. To call the parent constructor, we need
to call `this._super();` afterwards.

So lets imagine we have the following example:

```javascript
var util = require('util');
var moment = require('moment');
var mysql = require('mysql');
var Base = require('db-migrate-base');
var Promise = require('bluebird');
var log;
var type;

var internals = {};

var MysqlDriver = Base.extend({
  init: function(connection) {
    this._super(internals);
    this.connection = connection;
  },

  close: function(callback) {
    this.connection.end(callback);
  }

});

exports.connect = function(config, intern, callback) {
  var db;

  internals = intern;
  log = internals.mod.log;
  type = internals.mod.type;

  if (typeof(mysql.createConnection) === 'undefined') {
    db = config.db || new mysql.createClient(config);
  } else {
    db = config.db || new mysql.createConnection(config);
  }
  callback(null, new MysqlDriver(db));
};
```

#### Running sql

So you probably noticed I have defined a `log` and `type` *var* which are emtpy
and set the value of them in the exported `connect` function. These are two
functions/objects we get passed from db-migrate. `type` contains the standard
datatypes of *db-migrate* and `log` can be used to log messages on the info,
verbose, etc. channels. You may always prefer `log` over `console.log` as it
takes also care about the current settings within db-migrate. We also store
a reference of the internals in a local variable to access it later if needed.

In our constructor `init` we pass the internals to our parent base as the first
parameter and store the connection we got passed from our `connect`function
within the instance.
The next step is to override the close function, which either returns a promise
or calls the callback. We always provide both possibilities in db-migrate, you
should do this in your driver, too.

The next things we need to do, to have the very basic functionality, is to give
our driver the to actually control the migrations table, such as reading from, 
writing to, deleting from and creating the migrations table. So first of all,
we need to override the `all` and `runSql` methods.
`all` is similar to `runSql`, but runs always, even if dry-run is active!

```javascript
  runSql: function() {

    var self = this;
    var args = this._makeParamArgs(arguments);
    var callback = args.pop();
    log.sql.apply(null, arguments);
    if(internals.dryRun) {
      return callback();
    }

    return new Promise(function(resolve, reject) {
      args.push(function(err, data) {
        return (err ? reject(err) : resolve(data));
      });

      self.connection.query.apply(self.connection, args);
    }).nodeify(callback);
  },

  _makeParamArgs: function(args) {
    var params = Array.prototype.slice.call(args);
    var sql = params.shift();
    var callback = params.pop();

    if (params.length > 0 && Array.isArray(params[0])) {
      params = params[0];
    }
    return [sql, params, callback];
  },

  all: function() {
    var args = this._makeParamArgs(arguments);
    return this.connection.query.apply(this.connection, args);
  },
```

#### Provide migration capabilities

Now we have these, we need our migration methods. First of all, the one method
which needs always to be executed, even while a dry-run is running. The method
is called `allLoadedMigrations` and returns as the name says all already loaded
migrations.

**Attention! ALWAYS** provide a proper quoting through your driver, the 
community and db-migrate relies on having an automatic quoting!


```javascript
  /**
   * Queries the migrations table
   *
   * @param callback
   */
  allLoadedMigrations: function(callback) {
    var sql = 'SELECT * FROM `' + internals.migrationTable + '` ORDER BY run_on DESC, name DESC';
    this.all(sql, callback);
  },
```

Ok, now we can differenciate between already loaded migrations and not executed
ones. Great! Now we provide the ability to delete them `deleteMigration` and 
add them `addMigrationRecord`

```javascript
  addMigrationRecord: function (name, callback) {
    var formattedDate = moment(new Date()).format('YYYY-MM-DD HH:mm:ss');
    this.runSql('INSERT INTO `' + internals.migrationTable + '` (`name`, `run_on`) VALUES (?, ?)', [name, formattedDate], callback);
  },

  /**
   * Deletes a migration
   *
   * @param migrationName   - The name of the migration to be deleted
   * @param callback
   */
  deleteMigration: function(migrationName, callback) {
    var sql = 'DELETE FROM `' + internals.migrationTable + '` WHERE name = ?';
    this.runSql(sql, [migrationName], callback);
  },

  createTable: function(tableName, options, callback) {
    log.verbose('creating table:', tableName);
    var columnSpecs = options,
        tableOptions = {};

    if (options.columns !== undefined) {
      columnSpecs = options.columns;
      delete options.columns;
      tableOptions = options;
    }

    var ifNotExistsSql = "";
    if(tableOptions.ifNotExists) {
      ifNotExistsSql = "IF NOT EXISTS";
    }

    var primaryKeyColumns = [];
    var columnDefOptions = {
      emitPrimaryKey: false
    };

    for (var columnName in columnSpecs) {
      var columnSpec = this.normalizeColumnSpec(columnSpecs[columnName]);
      columnSpecs[columnName] = columnSpec;
      if (columnSpec.primaryKey) {
        primaryKeyColumns.push(columnName);
      }
    }

    var pkSql = '';
    if (primaryKeyColumns.length > 1) {
      pkSql = util.format(', PRIMARY KEY (`%s`)', primaryKeyColumns.join('`, `'));
    } else {
      columnDefOptions.emitPrimaryKey = true;
    }

    var columnDefs = [];
    var foreignKeys = [];

    for (var columnName in columnSpecs) {
      var columnSpec = columnSpecs[columnName];
      var constraint = this.createColumnDef(columnName, columnSpec, columnDefOptions, tableName);

      columnDefs.push(constraint.constraints);
      if (constraint.foreignKey)
        foreignKeys.push(constraint.foreignKey);
    }

    var sql = util.format('CREATE TABLE %s `%s` (%s%s)', ifNotExistsSql, tableName, columnDefs.join(', '), pkSql);

    return this.runSql(sql)
    .then(function()
    {

        return this.recurseCallbackArray(foreignKeys);
    }.bind(this)).nodeify(callback);
  },
```

You might noticed we also added the createTable method already. This is because
of the `createMigrationsTable` method, which utilize this method to create the
migrations table.
I will leave this aside for now, we will get here later again.

#### Example of basic driver

Ok, now we have a full functional driver. Currently we can only execute raw
sql within our migrations, if using this driver, but db-migrate has now all
functionality it needs to work properly. Our driver looks now like the 
following: 

**Note:** This is not a complete version of a working driver. Please
follow the tutorial until the end.

```javascrip
var util = require('util');
var moment = require('moment');
var mysql = require('mysql');
var Base = require('db-migrate-base');
var Promise = require('bluebird');
var log;
var type;

var internals = {};

var MysqlDriver = Base.extend({
  init: function(connection) {
    this._super(internals);
    this.connection = connection;
  },

  addMigrationRecord: function (name, callback) {
    var formattedDate = moment(new Date()).format('YYYY-MM-DD HH:mm:ss');
    this.runSql('INSERT INTO `' + internals.migrationTable + '` (`name`, `run_on`) VALUES (?, ?)', [name, formattedDate], callback);
  },

  runSql: function() {

    var self = this;
    var args = this._makeParamArgs(arguments);
    var callback = args.pop();
    log.sql.apply(null, arguments);
    if(internals.dryRun) {
      return callback();
    }

    return new Promise(function(resolve, reject) {
      args.push(function(err, data) {
        return (err ? reject(err) : resolve(data));
      });

      self.connection.query.apply(self.connection, args);
    }).nodeify(callback);
  },

  _makeParamArgs: function(args) {
    var params = Array.prototype.slice.call(args);
    var sql = params.shift();
    var callback = params.pop();

    if (params.length > 0 && Array.isArray(params[0])) {
      params = params[0];
    }
    return [sql, params, callback];
  },

  all: function() {
    var args = this._makeParamArgs(arguments);
    return this.connection.query.apply(this.connection, args);
  },

  /**
   * Queries the migrations table
   *
   * @param callback
   */
  allLoadedMigrations: function(callback) {
    var sql = 'SELECT * FROM `' + internals.migrationTable + '` ORDER BY run_on DESC, name DESC';
    this.all(sql, callback);
  },

  /**
   * Deletes a migration
   *
   * @param migrationName   - The name of the migration to be deleted
   * @param callback
   */
  deleteMigration: function(migrationName, callback) {
    var sql = 'DELETE FROM `' + internals.migrationTable + '` WHERE name = ?';
    this.runSql(sql, [migrationName], callback);
  },

  createTable: function(tableName, options, callback) {
    log.verbose('creating table:', tableName);
    var columnSpecs = options,
        tableOptions = {};

    if (options.columns !== undefined) {
      columnSpecs = options.columns;
      delete options.columns;
      tableOptions = options;
    }

    var ifNotExistsSql = "";
    if(tableOptions.ifNotExists) {
      ifNotExistsSql = "IF NOT EXISTS";
    }

    var primaryKeyColumns = [];
    var columnDefOptions = {
      emitPrimaryKey: false
    };

    for (var columnName in columnSpecs) {
      var columnSpec = this.normalizeColumnSpec(columnSpecs[columnName]);
      columnSpecs[columnName] = columnSpec;
      if (columnSpec.primaryKey) {
        primaryKeyColumns.push(columnName);
      }
    }

    var pkSql = '';
    if (primaryKeyColumns.length > 1) {
      pkSql = util.format(', PRIMARY KEY (`%s`)', primaryKeyColumns.join('`, `'));
    } else {
      columnDefOptions.emitPrimaryKey = true;
    }

    var columnDefs = [];
    var foreignKeys = [];

    for (var columnName in columnSpecs) {
      var columnSpec = columnSpecs[columnName];
      var constraint = this.createColumnDef(columnName, columnSpec, columnDefOptions, tableName);

      columnDefs.push(constraint.constraints);
      if (constraint.foreignKey)
        foreignKeys.push(constraint.foreignKey);
    }

    var sql = util.format('CREATE TABLE %s `%s` (%s%s)', ifNotExistsSql, tableName, columnDefs.join(', '), pkSql);

    return this.runSql(sql)
    .then(function()
    {

        return this.recurseCallbackArray(foreignKeys);
    }.bind(this)).nodeify(callback);
  },

  close: function(callback) {
    this.connection.end(callback);
  }

});

exports.connect = function(config, intern, callback) {
  var db;

  internals = intern;
  log = internals.mod.log;
  type = internals.mod.type;

  if (typeof(mysql.createConnection) === 'undefined') {
    db = config.db || new mysql.createClient(config);
  } else {
    db = config.db || new mysql.createConnection(config);
  }
  callback(null, new MysqlDriver(db));
};
```

#### Standard API

Now that we have a really basic driver, which is only capable of creating the
migrations table and adding/deleting migration records.

Now let us add some functionality of the standard api. Let's start with a 
question.

Does our driver support transactions?

No?

Ok, go ahead and skip to the next part. You don't need to do anything.

Yes?

Fine, the next chapter is just for you!

#### Transactions

To be able to start a transaction and commit it on the end of the migration,
our driver needs obviously the possibility to support this.

This is as simple as adding this two methods:

```javascript
  startMigration: function(cb){

    var self = this;

    if(!internals.notansactions) {

      return this.runSql('SET AUTOCOMMIT=0;')
             .then(function() {
                return self.runSql('START TRANSACTION;');
            })
             .nodeify(cb);
    }
    else
      return Promise.resolve().nodeify(cb);
  },

  endMigration: function(cb){

    if(!internals.notransactions) {

      return this.runSql('COMMIT;').nodeify(cb);
    }
    else
      return Promise.resolve(null).nodeify(cb);
  },
``` 

If you don't have transactions you can omit these methods, then the default no
transaction methods are going to be used.

#### Datatypes

Ok lets continue with adding the drivers interpretation of our datatypes. We
create a new method `mapDataType` and check wich value the spec.type value has
got. After we're done we call the parent method of this, which applies 
additional default behavior. Take a look at the base driver if you want to know
which exactly these are.

```javascript
  mapDataType: function(spec) {
    var len;
    switch(spec.type) {
      case type.TEXT:
        len = parseInt(spec.length, 10) || 1000;
        if(len > 16777216) {
          return 'LONGTEXT';
        }
        if(len > 65536) {
          return 'MEDIUMTEXT';
        }
        if(len > 256) {
          return 'TEXT';
        }
        return 'TINYTEXT';
      case type.DATE_TIME:
        return 'DATETIME';
      case type.BLOB:
        len = parseInt(spec.length, 10) || 1000;
        if(len > 16777216) {
          return 'LONGBLOB';
        }
        if(len > 65536) {
          return 'MEDIUMBLOB';
        }
        if(len > 256) {
          return 'BLOB';
        }
        return 'TINYBLOB';
      case type.BOOLEAN:
        return 'TINYINT(1)';
    }
    return this._super(spec.type);
  },
```

#### The real driver

Now that we have the basic driver, it is time to create a concept for our
new driver.
First of all, what are we going to do? Are we creating a driver for a
relational schema, or a NoSQL one? If we're go for the relational one we
need capabilities like foreignKeys, when we go for the NoSQL one we don't.

You can get some orientation for both by viewing existing drivers, like the one
for mysql, or the one for mongodb. But you also have to determine what features
you need to be supported. Check if this is met by the standardized requirements
of the API you're currently writing your driver for. Or if you need, or want
additional behavior. If the latter is the case, you also need to provide your
own documentation. Otherwise the users wont know about what your driver do, 
also the db-migrate project wont ever accept your driver as an official one if
it is not documented.

At least you have to follow one rule:
Always provide the complete capability which is defined by the standard API of
the Schema you're working on. For example you always need to provide an
addColumn method, on the other side there are things you can provide, but you
don't have to. For example transactions, if you don't have transactions, you
can't implement them, obviously. However, if you can't support a feature
defined by this standard API, thats often the case for transactions for example
you should outline this in your documentation, so the user is warned about it.

#### Publishing your driver

Ok, you're finally this far?
Great! You did some awesome work and now you want to publish it. But how to do
so?

It's as easy as publishing a normal npm module, the only requirment is the
name! To make other users to be able to use your driver, first choose a name
for your driver. You have one? Ok, now go to npm and check if your driver name
was already taken, or if you can use it. Search with the term db-migrate-name.
If it is not taken, then name your project in your package.json with your name
and the db-migrate- prefix and publish it. And you're done, now users that
install your driver by

    $ npm install -g db-migrate-awesome

can use it from that point. Your driver would be available from the config with
your name without the prefix. So you configure as name awesome, not 
db-migrate-awesome.

#### Finish?

So are we done yet? Well no, we aren't. You have a new driver, but an untested
one. It is going to happen, what you've done is not completely free from bugs.
So rule number one is, **write tests**!

So how to test our driver? A good start would be to take a look at existing 
tests of other drivers. They're bascially much the same and have small
differences, like datatypes, return types and so on. Most of the time you
should be able to take a copy of an existing test suite and make changes to it
to ensure completeness and validity.
You're also free to choose the framework which you're going to use when testing 
your driver, to note offical drivers use `vows` or `lab`.

#### Maintaining your driver

Ok finally we're done with that kind of things. Our driver is now tested, and
probably it works like it should. Now we can wait for user feedback and we all
know this is the greatest thing about open source. The community reports you
bugs you didn't mentoined and also come with ideas, to improve your driver.
Step by step your driver becomes more powerful, but keep the next hints in 
mind.

If a user opens an issue to propagate his idea about a maybe awesome feature,
always keep in mind, your driver wont be accepted anymore if you loose the
focus. We have a migrator and a seeder, so if the user want a feature think
about, where does it fit. Does it even fit or does this have nothing todo with
the migrator itself. Sometimes you have to decline feature requests or even
pull requests, because they doesn't fit or actually have nothing to do with
each other.

You say this sounds weird?
Yes it is, but do we want a bloated API that covers everything even if this
doesn't belongs to what the original product is described of?
Clearly not, following this advices doesn't only help you to make your driver
an offical accepted one, it also helps keeping the project structured.
