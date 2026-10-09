# Migrations API - NoSQL

The operations of the [MongoDB driver](../Drivers/mongodb.md) inside a v1
migration. Every operation returns a promise and takes an optional callback as
last argument, unless noted otherwise.

```javascript
exports.up = async function (db) {
  await db.createCollection('pets');
  await db.addIndex('pets', 'pets_name', ['name'], true);
  await db.insert('pets', [{ name: 'Rex' }, { name: 'Tom' }]);
};

exports.down = function (db) {
  return db.dropCollection('pets');
};
```

### createCollection(collectionName, [callback])

Creates a new collection.

__Arguments__

* collectionName - the name of the collection to create
* callback(err) - callback that will be invoked after creating the collection

### dropCollection(collectionName, [callback])

Drops a collection.

__Arguments__

* collectionName - name of the collection to drop
* callback(err) - callback that will be invoked after dropping the collection

### renameCollection(collectionName, newCollectionName, [callback])

Renames a collection.

__Arguments__

* collectionName - existing collection name
* newCollectionName - new collection name
* callback(err) - callback that will be invoked after renaming the collection

### addIndex(collectionName, indexName, columns, unique, [callback])

Adds an index.

__Arguments__

* collectionName - collection to add the index to
* indexName - the name of the index
* columns - the fields of the index, passed on to MongoDB's `createIndex`
* unique - whether the index is unique
* callback(err) - callback that will be invoked after adding the index

### removeIndex(collectionName, indexName, [callback])

Removes an index.

__Arguments__

* collectionName - name of the collection that has the index
* indexName - the name of the index
* callback(err) - callback that will be invoked after removing the index

### insert(collectionName, toInsert, [callback])

Inserts documents into a collection.

__Arguments__

* collectionName - collection to insert into
* toInsert - a document, or an array of documents
* callback(err) - callback that will be invoked once the insert has been completed

### createTable, dropTable and renameTable

`createTable(collectionName, callback)` and `dropTable(collectionName,
callback)` are aliases of `createCollection` and `dropCollection` which only
work with a callback, they return no promise. Use `createCollection` and
`dropCollection` with promises. `renameTable` is an alias of
`renameCollection`.
