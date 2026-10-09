# Generic Datatypes

Generic data types make migrations more database independent. They are
available as `dbm.dataType` in `setup` of a v1 migration, as
`opt.dbm.dataType` in a v2 migration, and as `dataType` of the
[programmable API](programable.md).

```javascript
var type;

exports.setup = function (options) {
  type = options.dbmigrate.dataType;
};

exports.up = function (db) {
  return db.createTable('pets', {
    id: { type: type.INTEGER, primaryKey: true },
    name: { type: type.STRING, length: 64 }
  });
};
```

| Constant | Value |
|---|---|
| `CHAR` | `char` |
| `STRING` | `string` |
| `TEXT` | `text` |
| `SMALLINT`, `SMALL_INTEGER` | `smallint` |
| `INTEGER` | `int` |
| `BIGINT`, `BIG_INTEGER` | `bigint` |
| `REAL` | `real` |
| `DECIMAL` | `decimal` |
| `BOOLEAN` | `boolean` |
| `DATE` | `date` |
| `TIME` | `time` |
| `DATE_TIME` | `datetime` |
| `TIMESTAMP` | `timestamp` |
| `BLOB` | `blob` |
| `BINARY` | `binary` |

The values can be used directly as well, `type: 'string'`. Most types map to
the SQL type of the same name, the exceptions are listed on the driver pages:
[PostgreSQL](../Drivers/pg.md#data-types), [MySQL](../Drivers/mysql.md#data-types),
[sqlite3](../Drivers/sqlite3.md#data-types),
[CockroachDB](../Drivers/cockroachdb.md#data-types). `string` becomes
`VARCHAR` everywhere.

Any other type is passed to the database as it is, upper cased, with a
warning, so database specific types like `uuid` work as well.
