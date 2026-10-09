# MySQL

    $ npm install db-migrate-mysql

Based on [mysql2](https://github.com/sidorares/node-mysql2), for MySQL and
MariaDB.

## Configuration

```json
{
  "dev": {
    "driver": "mysql",
    "host": "localhost",
    "user": { "ENV" : "DB_USER" },
    "password" : { "ENV" : "DB_PASS" },
    "database": "database-name",
    "multipleStatements": true
  }
}
```

The settings, except `driver`, are passed to `mysql2.createConnection`, so
every connection option of mysql2 works, like `port`, `ssl` or `charset`.

### multipleStatements

To run several statements with one `runSql`, e.g. from sql files created with
`--sql-file` or with [db-migrate-plugin-sql](../Getting Started/plugins.md),
set `multipleStatements: true`.

## Data types

| Type | MySQL |
|---|---|
| `string` | `VARCHAR`, with length 255 if no `length` is given |
| `text` | `TINYTEXT`, `TEXT`, `MEDIUMTEXT` or `LONGTEXT`, chosen by `length` (default 1000, so `TEXT`) |
| `blob` | `TINYBLOB`, `BLOB`, `MEDIUMBLOB` or `LONGBLOB`, chosen by `length` |
| `datetime` | `DATETIME` |
| `boolean` | `TINYINT(1)` |
| `json` | `JSON` |
| `decimal` | `DECIMAL(precision,scale)` with `precision` and `scale` set |

The other [generic types](../API/generic datatypes.md) map to the SQL type of
the same name, any other type is passed on as is.

## Column spec

In addition to the [common column spec](../API/SQL.md#column-spec):

| Option | |
|---|---|
| `unsigned` | `UNSIGNED` |
| `characterSet` | `CHARACTER SET` of the column |
| `collation` | `COLLATE` of the column |
| `onUpdate` | `ON UPDATE`, only values starting with `CURRENT_TIMESTAMP` |
| `null` | `true` to mark the column as `NULL` explicitly, like `notNull: false` |
| `after` | add the column after the given column |
| `comment` | `COMMENT` |
| `precision`, `scale` | for `decimal` |

A `defaultValue` starting with `CURRENT_TIMESTAMP` is not quoted,
`defaultValue: null` sets `DEFAULT NULL`.

```js
db.addColumn('pets', 'updated_at', {
  type: 'timestamp',
  defaultValue: 'CURRENT_TIMESTAMP',
  onUpdate: 'CURRENT_TIMESTAMP',
  after: 'name'
});
```

## Table options

`createTable` with a `columns` object takes:

| Option | |
|---|---|
| `engine` | `ENGINE`, e.g. `InnoDB` |
| `rowFormat` | `ROW_FORMAT` |
| `charset` | `CHARACTER SET` |
| `collate` | `COLLATE` |

```js
db.createTable('pets', {
  columns: {
    id: { type: 'int', primaryKey: true, autoIncrement: true },
    name: 'string'
  },
  engine: 'InnoDB',
  charset: 'utf8mb4'
});
```

## Further notes

- `addIndex` takes columns as names, or as `{ name, length }` for prefix
  indexes: `db.addIndex('pets', 'pets_name', [{ name: 'name', length: 10 }])`.
- `removeIndex` requires the table name.
- `removeForeignKey` takes `{ dropIndex: true }` to drop the index of the
  same name as well.
- `renameColumn` keeps the type of the column.
- `changeColumn` redefines the column with the given spec
  (`CHANGE COLUMN`), `unique: false` drops the index named like the column.
- `all` only works with a callback, it does not return a promise. Use
  `runSql` with a promise instead, it resolves with the rows of a query.
- `db:create` and `db:drop` use `IF NOT EXISTS` and `IF EXISTS`.
- A [scope](../Getting Started/commands.md#scope-configuration) `config.json`
  switches the database with `USE`, it has to contain `database`.
- v1 migrations run inside a transaction (`START TRANSACTION` ... `COMMIT`),
  unless `--non-transactional`. Note that MySQL commits most schema changes
  implicitly.
- No [column strategies](../Guides/migrations v2.md#removing-notnull-columns),
  so v2 migrations can not remove `notNull` columns.
- Supports the [migration lock](../Guides/running in parallel.md) since 3.1.0.
