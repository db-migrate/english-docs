## Migrations API - NoSQL

Below are examples of all the different migrations supported by db-migrate for NoSQL databases.

### createCollection(collectionName, callback)

Creates a new collection.

__Arguments__

* collectionName - the name of the collection to create
* callback(err) - callback that will be invoked after table creation

__Examples__

```javascript
exports.up = function (db, callback) {
  db.createCollection('pets', callback);
}
```

### dropCollection(collectionName, callback)

Drop a database collection

__Arguments__

* collectionName - name of the collection to drop
* callback(err) - callback that will be invoked after dropping the collection

### renameCollection(collectionName, newCollectionName, callback)

Rename a database table

__Arguments__

* collectionName - existing collection name
* newCollectionName - new collection name
* callback(err) - callback that will be invoked after renaming the collection

### addIndex(collectionName, indexName, columns, unique, callback)

Add an index

__Arguments__

* collectionName - collection to add the index too
* indexName - the name of the index
* columns - an array of column names contained in the index
* unique - whether the index is unique
* callback(err) - callback that will be invoked after adding the index

### removeIndex(collectionName, indexName, callback)

Remove an index

__Arguments__

* collectionName - name of the collection that has the index
* indexName - the name of the index
* callback(err) - callback that will be invoked after removing the index

### insert(collectionName, toInsert, callback)

Insert an item into a given collection

__Arguments__

* collectionName - collection to insert the item into
* toInsert - an object or array of objects to be inserted into the associated collection
* callback(err) - callback that will be invoked once the insert has been completed.
