## Migrations API - SQL

Below are examples of all the different migrations supported by db-migrate. Please note that not all migrations are supported by all databases. For example, SQLite does not support dropping columns.

### createTable(tableName, columnSpec, callback)

Creates a new table with the specified columns.

__Arguments__

* tableName - the name of the table to create
* columnSpec - a hash of column definitions
* callback(err) - callback that will be invoked after table creation

__Examples__

```javascript
// with no table options
exports.up = function (db, callback) {
  db.createTable('pets', {
    id: { type: 'int', primaryKey: true, autoIncrement: true },
    name: 'string'  // shorthand notation
  }, callback);
}

// with table options
exports.up = function (db, callback) {
  db.createTable('pets', {
    columns: {
      id: { type: 'int', primaryKey: true, autoIncrement: true },
      name: 'string'  // shorthand notation
    },
    ifNotExists: true
  }, callback);
}
```

__Column Specs__

The following options are available on column specs

* type - the column data type. Supported types can be found in lib/data_type.js
* length - the column data length, where supported
* primaryKey - true to set the column as a primary key. Compound primary keys are supported by setting the `primaryKey` option to true on multiple columns
* autoIncrement - true to mark the column as auto incrementing
* notNull - true to mark the column as non-nullable, omit it archive database default behavior and false to mark explicitly as nullable
* unique - true to add unique constraint to the column
* defaultValue - set the column default value
* foreignKey - set a foreign key to the column

__Column ForeignKey Spec Examples__

**Note:** Currently only supported together with mysql!

```javascript
exports.up = function(db, callback) {

  //automatic mapping, the mapping key resolves to the column
  db.createTable( 'product_variant',
  {
      id:
      {
        type: 'int',
        unsigned: true,
        notNull: true,
        primaryKey: true,
        autoIncrement: true,
        length: 10
      },
      product_id:
      {
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
      },
  }, callback );
};

exports.up = function(db, callback) {

  //explicit mapping
  db.createTable( 'product_variant',
  {
    id:
    {
      type: 'int',
      unsigned: true,
      notNull: true,
      primaryKey: true,
      autoIncrement: true,
      length: 10
    },
    product_id:
    {
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
    },
  }, callback );
};
```

### dropTable(tableName, [options,] callback)

Drop a database table

__Arguments__

* tableName - name of the table to drop
* options - table options
* callback(err) - callback that will be invoked after dropping the table

__Table Options__

* ifExists - Only drop the table if it already exists

### renameTable(tableName, newTableName, callback)

Rename a database table

__Arguments__

* tableName - existing table name
* options - new table name
* callback(err) - callback that will be invoked after renaming the table

### addColumn(tableName, columnName, columnSpec, callback)

Add a column to a database table

__Arguments__

* tableName - name of table to add a column to
* columnName - name of the column to add
* columnSpec - a hash of column definitions
* callback(err) - callback that will be invoked after adding the column

Column spec is the same as that described in createTable

### removeColumn(tableName, columnName, callback)

Remove a column from an existing database table

* tableName - name of table to remove a column from
* columnName - name of the column to remove
* callback(err) - callback that will be invoked after removing the column

### renameColumn(tableName, oldColumnName, newColumnName, callback)

Rename a column

__Arguments__

* tableName - table containing column to rename
* oldColumnName - existing column name
* newColumnName - new name of the column
* callback(err) - callback that will be invoked after renaming the column

### changeColumn(tableName, columnName, columnSpec, callback)

Change the definition of a column

__Arguments__

* tableName - table containing column to change
* columnName - existing column name
* columnSpec - a hash containing the column spec
* callback(err) - callback that will be invoked after changing the column

### addIndex(tableName, indexName, columns, [unique], callback)

Add an index

__Arguments__

* tableName - table to add the index too
* indexName - the name of the index
* columns - an array of column names contained in the index
* unique - whether the index is unique (optional, default false)
* callback(err) - callback that will be invoked after adding the index

### addForeignKey

Adds a foreign Key

__Arguments__

* tableName - table on which the foreign key gets applied
* referencedTableName - table where the referenced key is located
* keyName - name of the foreign key
* fieldMapping - mapping of the foreign key to referenced key
* rules - ondelete, onupdate constraints
* callback(err) - callback that will be invoked after adding the foreign key

__Example__

```javascript
exports.up = function (db, callback)
{
  db.addForeignKey('module_user', 'modules', 'module_user_module_id_foreign',
  {
    'module_id': 'id'
  },
  {
    onDelete: 'CASCADE',
    onUpdate: 'RESTRICT'
  }, callback);
};
```

### removeForeignKey

__Arguments__

* tableName - table in which the foreign key should be deleted
* keyName - the name of the foreign key
* options - object of options, see below
* callback - callback that will be invoked once the foreign key was deleted

__Options__

* dropIndex (default: false) - deletes the index with the same name as the foreign key

__Examples__

```javascript
//without options object
exports.down = function (db, callback)
{
  db.removeForeignKey('module_user', 'module_user_module_id_foreign', callback);
};

//with options object
exports.down = function (db, callback)
{
  db.removeForeignKey('module_user', 'module_user_module_id_foreign',
  {
    dropIndex: true,
  }, callback);
};
```

### insert(tableName, columnNameArray, valueArray, callback)

Insert an item into a given column

__Arguments__

* tableName - table to insert the item into
* columnNameArray - the array existing column names for each item being inserted
* valueArray - the array of values to be inserted into the associated column
* callback(err) - callback that will be invoked once the insert has been completed.

### removeIndex([tableName], indexName, callback)

Remove an index

__Arguments__

* tableName - name of the table that has the index (Required for mySql)
* indexName - the name of the index
* callback(err) - callback that will be invoked after removing the index

### runSql(sql, [params,] callback)

Run arbitrary SQL

__Arguments__

* sql - the SQL query string, possibly with ? replacement parameters
* params - zero or more ? replacement parameters
* callback(err) - callback that will be invoked after executing the SQL

### all(sql, [params,] callback)

Execute a select statement, even in dry run mode. Attention, only use this if
you know what you're doing. This can cause you issues if you're utilizing
the dry-run mode for testings. To execute sql queries always use runSql!

__Arguments__

* sql - the SQL query string, possibly with ? replacement parameters
* params - zero or more ? replacement parameters
* callback(err, results) - callback that will be invoked after executing the SQL
