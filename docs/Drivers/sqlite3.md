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
| `filename` | the database file, required, without it connecting fails (since 1.1.1, before the run hung). It is created if it does not exist. |
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
`removeColumn`, `renameColumn`, `addIndex`, `removeIndex`, `insert`, `runSql`
and `all` of the [SQL API](../API/SQL.md). `removeColumn` and `renameColumn`
need db-migrate-sqlite3 1.2.0. SQLite can not drop a column that is part of
the primary key, unique, indexed or referenced by a foreign key, remove the
index first.

Not implemented, they fail with `not implemented`: `changeColumn`,
`addForeignKey`, `removeForeignKey`. A `foreignKey` in a column spec is
ignored. Run the SQL yourself with `runSql` for these.

`autoIncrement` on a single primary key creates `PRIMARY KEY AUTOINCREMENT`.
`defaultValue: { special: 'CURRENT_TIMESTAMP' }` sets the default to
`CURRENT_TIMESTAMP`.

## Further notes

- `db:create` succeeds without doing anything, sqlite creates the file on
  its own when connecting (since 1.1.1). `db:drop` fails with
  `sqlite has no databases to drop, delete the database file instead`.
- A [scope](../Getting Started/commands.md#scope-configuration) `config.json`
  with only `database` or `schema` is ignored. One with a `filename` connects
  the scope to that file, with its own migrations and state tables.
- v1 migrations run inside `BEGIN TRANSACTION` ... `COMMIT`, unless
  `--non-transactional`.
- v2 migrations can only use the instructions implemented by the driver, and
  their reverse operations need them as well, e.g. reverting an `addColumn`
  needs `removeColumn`, available since 1.2.0.
- Supports the [migration lock](../Guides/running in parallel.md) since 1.1.0.
