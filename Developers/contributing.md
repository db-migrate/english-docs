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

2. For the repo.

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

  * Create your own driver

  * Contribute to db-migrate itself


### Contributing to db-migrate

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

### Creating your own driver

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
