# Migrations API - SQL

The operations of the SQL drivers inside a migration. Not every driver
supports every operation, see the overview at the end and the
[driver pages](../drivers.md) for their additions.

Every operation returns a promise and takes an optional callback as last
argument. In [v2 migrations](../Guides/migrations v2.md) only promises work,
and only the instructions db-migrate can revert, `insert`, `runSql` and `all`
are not available there.

### createTable(tableName, columnSpec, [callback])

Creates a new table with the specified columns.

__Arguments__

* tableName - the name of the table to create
* columnSpec - a hash of column definitions, or an object with the columns in
  `columns` and table options
* callback(err) - callback that will be invoked after table creation

__Examples__

```javascript
// with no table options
exports.up = function (db) {
  return db.createTable('pets', {
    id: { type: 'int', primaryKey: true, autoIncrement: true },
    name: 'string'  // shorthand notation
  });
};

// with table options
exports.up = function (db) {
  return db.createTable('pets', {
    columns: {
      id: { type: 'int', primaryKey: true, autoIncrement: true },
      name: 'string'  // shorthand notation
    },
    ifNotExists: true
  });
};
```

__Table Options__

* ifNotExists - only create the table if it does not exist yet

Drivers add their own, see [MySQL](../Drivers/mysql.md#table-options) and
[CockroachDB](../Drivers/cockroachdb.md#table-options).

### Column spec

A column is either a type, like `name: 'string'`, or an object:

* type - the column data type, see [Generic Datatypes](generic datatypes.md)
* length - the column data length, where supported
* primaryKey - true to set the column as a primary key. Compound primary keys
  are supported by setting `primaryKey` on multiple columns
* autoIncrement - true to mark the column as auto incrementing
* notNull - true to mark the column as non-nullable, omit it for the database
  default behavior
* unique - true to add a unique constraint to the column
* defaultValue - the default value of the column, see below
* foreignKey - a foreign key, see below

Drivers support further options, see their pages.

__Default values__

* a string is quoted: `defaultValue: 'none'`
* numbers and booleans are used as they are: `defaultValue: 0`
* an expression, like a function call, is used unchanged with
  `defaultValue: { raw: 'uuid_generate_v4()' }` (mysql since 3.1.1), or passed
  as a String object: `defaultValue: new String('uuid_generate_v4()')`
* the current time: `defaultValue: { special: 'CURRENT_TIMESTAMP' }`
  (pg, sqlite3, cockroachdb, mysql since 3.1.1)

A special default value the driver does not support is dropped with a
warning (db-migrate-base 2.4.1 with pg 1.6.1, sqlite3 1.1.1, cockroachdb
5.8.2, mysql 3.1.1; older versions fail).

__Foreign keys__

**Note:** Supported by pg, mysql and cockroachdb, sqlite3 ignores them.

* name - the name of the foreign key
* table - the referenced table
* mapping - the referenced column, or an object mapping columns of this table
  to columns of the referenced table
* rules - `onDelete` and `onUpdate`, `NO ACTION` by default

```javascript
exports.up = function(db) {

  //automatic mapping, the mapping key resolves to the column
  return db.createTable('product_variant', {
    id: {
      type: 'int',
      unsigned: true,
      notNull: true,
      primaryKey: true,
      autoIncrement: true,
      length: 10
    },
    product_id: {
      type: 'int',
      unsigned: true,
      length: 10,
      notNull: true,
      foreignKey: {
        name: 'product_variant_product_id_fk',
        table: 'product',
        rules: {
          onDelete: 'CASCADE',
          onUpdate: 'RESTRICT'
        },
        mapping: 'id'
      }
    }
  });
};

exports.up = function(db) {

  //explicit mapping
  return db.createTable('product_variant', {
    id: {
      type: 'int',
      unsigned: true,
      notNull: true,
      primaryKey: true,
      autoIncrement: true,
      length: 10
    },
    product_id: {
      type: 'int',
      unsigned: true,
      length: 10,
      notNull: true,
      foreignKey: {
        name: 'product_variant_product_id_fk',
        table: 'product',
        rules: {
          onDelete: 'CASCADE',
          onUpdate: 'RESTRICT'
        },
        mapping: {
          product_id: 'id'
        }
      }
    }
  });
};
```

### dropTable(tableName, [options], [callback])

Drop a database table

__Arguments__

* tableName - name of the table to drop
* options - table options
* callback(err) - callback that will be invoked after dropping the table

__Table Options__

* ifExists - Only drop the table if it already exists

### renameTable(tableName, newTableName, [callback])

Rename a database table

__Arguments__

* tableName - existing table name
* newTableName - new table name
* callback(err) - callback that will be invoked after renaming the table

### addColumn(tableName, columnName, columnSpec, [callback])

Add a column to a database table

__Arguments__

* tableName - name of table to add a column to
* columnName - name of the column to add
* columnSpec - a hash of column definitions, see [Column spec](#column-spec)
* callback(err) - callback that will be invoked after adding the column

### removeColumn(tableName, columnName, [callback])

Remove a column from an existing database table

__Arguments__

* tableName - name of table to remove a column from
* columnName - name of the column to remove
* callback(err) - callback that will be invoked after removing the column

pg and cockroachdb take an options object in place of the callback, see
[removing notNull columns](../Guides/migrations v2.md#removing-notnull-columns).

### renameColumn(tableName, oldColumnName, newColumnName, [callback])

Rename a column

__Arguments__

* tableName - table containing column to rename
* oldColumnName - existing column name
* newColumnName - new name of the column
* callback(err) - callback that will be invoked after renaming the column

### changeColumn(tableName, columnName, columnSpec, [callback])

Change the definition of a column

__Arguments__

* tableName - table containing column to change
* columnName - existing column name
* columnSpec - a hash containing the column spec
* callback(err) - callback that will be invoked after changing the column

How the spec is applied differs between the drivers, see
[PostgreSQL](../Drivers/pg.md#changecolumn),
[MySQL](../Drivers/mysql.md#further-notes) and
[CockroachDB](../Drivers/cockroachdb.md#changecolumn).

### addIndex(tableName, indexName, columns, [unique], [callback])

Add an index

__Arguments__

* tableName - table to add the index to
* indexName - the name of the index
* columns - an array of column names contained in the index
* unique - whether the index is unique (optional, default false)
* callback(err) - callback that will be invoked after adding the index

### removeIndex([tableName], indexName, [callback])

Remove an index

__Arguments__

* tableName - name of the table that has the index (required for mysql)
* indexName - the name of the index
* callback(err) - callback that will be invoked after removing the index

### addForeignKey(tableName, referencedTableName, keyName, fieldMapping, rules, [callback])

Adds a foreign key

__Arguments__

* tableName - table on which the foreign key gets applied
* referencedTableName - table where the referenced key is located
* keyName - name of the foreign key
* fieldMapping - mapping of the columns of the foreign key to the referenced
  columns
* rules - `onDelete` and `onUpdate`, `NO ACTION` by default
* callback(err) - callback that will be invoked after adding the foreign key

__Example__

```javascript
exports.up = function (db) {
  return db.addForeignKey('module_user', 'modules', 'module_user_module_id_foreign',
  {
    'module_id': 'id'
  },
  {
    onDelete: 'CASCADE',
    onUpdate: 'RESTRICT'
  });
};
```

### removeForeignKey(tableName, keyName, [options], [callback])

__Arguments__

* tableName - table in which the foreign key should be deleted
* keyName - the name of the foreign key
* options - object of options, see below
* callback - callback that will be invoked once the foreign key was deleted

__Options__

* dropIndex (default: false, mysql only) - deletes the index with the same
  name as the foreign key

__Examples__

```javascript
//without options object
exports.down = function (db) {
  return db.removeForeignKey('module_user', 'module_user_module_id_foreign');
};

//with options object
exports.down = function (db) {
  return db.removeForeignKey('module_user', 'module_user_module_id_foreign',
  {
    dropIndex: true,
  });
};
```

### insert(tableName, columnNameArray, valueArray, [callback])

Insert a row into a table

__Arguments__

* tableName - table to insert the row into
* columnNameArray - the names of the columns
* valueArray - the values, in the order of the columns
* callback(err) - callback that will be invoked once the insert has been completed

```javascript
exports.up = function (db) {
  return db.insert('pets', ['name', 'kind'], ['Rex', 'dog']);
};
```

### runSql(sql, [params], [callback])

Run arbitrary SQL

__Arguments__

* sql - the SQL query string, possibly with `?` replacement parameters
* params - an array of replacement parameters
* callback(err) - callback that will be invoked after executing the SQL

```javascript
exports.up = function (db) {
  return db.runSql('UPDATE pets SET kind = ? WHERE kind IS NULL', ['unknown']);
};
```

In a dry run (`--dry-run`) the SQL is only printed.

### all(sql, [params], callback)

Execute a select statement, even in dry run mode. Attention, only use this if
you know what you're doing. This can cause you issues if you're utilizing
the dry-run mode for testings. To execute sql queries always use runSql!

__Arguments__

* sql - the SQL query string
* params - an array of replacement parameters
* callback(err, results) - callback that will be invoked with the rows

With pg and cockroachdb, write the parameters as `$1`, `$2`, ... Before
db-migrate-mysql 3.1.1, `all` only worked with a callback there.

## Support by driver

| | pg | mysql | sqlite3 | cockroachdb |
|---|---|---|---|---|
| createTable, dropTable, renameTable, addColumn | yes | yes | yes | yes |
| removeColumn, renameColumn, changeColumn | yes | yes | no | yes |
| addIndex, removeIndex | yes | yes | yes | yes |
| addForeignKey, removeForeignKey, `foreignKey` | yes | yes | no | yes |
| insert, runSql, all | yes | yes | yes | yes |
