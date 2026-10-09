# sqlite3

    $ npm install db-migrate-sqlite3

Based on [node-sqlite3](https://github.com/TryGhost/node-sqlite3).

## Configuration

```json
{
  "dev": {
    "driver": "sqlite3",
    "filename": "dev.db"
  },
  "test": {
    "driver": "sqlite3",
    "filename": ":memory:"
  }
}
```

| Setting | |
|---|---|
| `filename` | the database file, required. It is created if it does not exist. |
| `mode` | the open mode of node-sqlite3, a number, by default `OPEN_READWRITE \| OPEN_CREATE` |

## Data types

| Type | SQLite |
|---|---|
| `datetime` | `datetime` |
| `time` | `time` |
| `int` | `INTEGER`, `length` is ignored |

The other [generic types](../API/generic datatypes.md) map to the SQL type of
the same name, any other type is passed on as is.

## Supported operations

The driver implements `createTable`, `dropTable`, `renameTable`, `addColumn`,
`addIndex`, `removeIndex`, `insert`, `runSql` and `all` of the
[SQL API](../API/SQL.md).

Not implemented, they fail with `not implemented`: `removeColumn`,
`renameColumn`, `changeColumn`, `addForeignKey`, `removeForeignKey`. A
`foreignKey` in a column spec is ignored. Run the SQL yourself with `runSql`
for these.

`autoIncrement` on a single primary key creates `PRIMARY KEY AUTOINCREMENT`.
`defaultValue: { special: 'CURRENT_TIMESTAMP' }` sets the default to
`CURRENT_TIMESTAMP`.

## Further notes

- `db:create` does nothing, the file is created on connect. `db:drop` is not
  supported.
- A [scope](../Getting Started/commands.md#scope-configuration) `config.json`
  is ignored.
- v1 migrations run inside `BEGIN TRANSACTION` ... `COMMIT`, unless
  `--non-transactional`.
- v2 migrations can only use the instructions implemented by the driver, and
  their reverse operations need them as well, e.g. reverting an `addColumn`
  needs `removeColumn`.
- Supports the [migration lock](../Guides/running in parallel.md) since 1.1.0.
