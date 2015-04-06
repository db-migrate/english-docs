# Programable API

This API is intended for the usage with db-migrate as a module and consists out
of several functions.

## Using db-migrate as module

To get an instance of db-migrate you need to call it in your application like
the following:

```javascript
var DBMigrate = require('db-migrate');
var assert = require('assert');

//getting an instance of dbmigrate
var dbmigrate_1 = DBMigrate.getInstance(true);

function specialCallback(migrator, originalError) {
  migrator.driver.close(function(err) {
    assert.ifError(originalErr);
    assert.ifError(err);
    log.info('Done');
  });
}

//specify an own callback, to handle errors on your side of the application.
var dbmigrate_2 = DBMigrate.getInstance(true, specialCallback);


//get an instance and call the standard runtime behavior
var dbmigrate_3 = DBMigrate.getInstance();
dbmigrate_3.run();
```