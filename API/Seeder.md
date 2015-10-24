# Seeder API

### insert(tableName, valueArray, callback)

Inserts a new set of data.

```javascript
exports.up = function( db ) {

  return db.insert( 'table', [
    { name: 'somename', surname: 'someSurname' },
    { name: 'anothername', surname: 'anotherSurname' }
  ] );
};
```

### remove(table, ids, callback)

Removes given datasets by their id.

```javascript
exports.up = function( db ) {

  return db.remove( 'table', { name: 'somename' } );
};
```

### update(tableName, columnNameArray, valueArray, ids, callback)

Updates one or multiple datasets within a specified table.

```javascript
exports.up = function( db ) {

  return db.update( 'table', { name: 'newname' }, { name: 'somename' } );
};
```

### lookup(tableName, column, id, callback)

Retrieves a value from the database, given the table, column and the id.
