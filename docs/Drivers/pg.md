# PostgreSQL

    $ npm install db-migrate-pg

Based on [node-postgres](https://node-postgres.com).

## Configuration

```json
{
  "dev": {
    "driver": "pg",
    "host": "localhost",
    "port": 5432,
    "user": "app",
    "password": {"ENV": "PGPASSWORD"},
    "database": "app"
  }
}
```

The settings are passed to `pg.Client`, so every
[option of node-postgres](https://node-postgres.com/apis/client) works.
`database` defaults to `postgres`.

| Setting | |
|---|---|
| `ssl` | `true`, or an object passed to node-postgres. With `sslmode` set in it, the files named by `sslrootcert`, `sslcert` and `sslkey` are read into `ca`, `cert` and `key`. |
| `native` | `true` to use the native bindings, `pg-native` has to be installed. |
| `schema` | see below |

```json
{
  "prod": {
    "driver": "pg",
    "host": "db.example.com",
    "database": "app",
    "ssl": {
      "sslmode": "verify-full",
      "sslrootcert": "/etc/ssl/certs/db-ca.pem"
    }
  }
}
```

### Schema

`schema` puts the given schema in front of the `search_path` of every
connection, so the migrations table, the state table and the tables of your
migrations land in it. The schema has to exist.

```json
{
  "dev": {
    "driver": "pg",
    "database": "app",
    "schema": "my_schema"
  }
}
```

**Note:** db-migrate-pg 1.6.0 does not apply `schema`, everything ends up in
the default `search_path`; update to 1.6.1. With 1.6.0, set the `search_path`
of the connection instead:

```json
{
  "dev": {
    "driver": "pg",
    "database": "app",
    "options": "-c search_path=my_schema"
  }
}
```

A [scope](../Getting Started/commands.md#scope-configuration) `config.json`
with only `schema` (or `database`) sets the `search_path` before the
migrations of the scope run. A scope `config.json` with further connection
settings connects on its own, there `database` is the real database.

## Data types

| Type | PostgreSQL |
|---|---|
| `string` | `VARCHAR` |
| `datetime` | `TIMESTAMP` |
| `blob` | `BYTEA` |
| `json`, `jsonb` | `JSON`, `JSONB` |
| `timestamptz`, `timetz` | `TIMESTAMP WITH TIME ZONE`, `TIME WITH TIME ZONE` |

The other [generic types](../API/generic datatypes.md) map to the SQL type of
the same name, any other type is passed on as is.

## Column spec

In addition to the [common column spec](../API/SQL.md#column-spec):

| Option | |
|---|---|
| `comment` | adds a comment to the column |

`autoIncrement` on a primary key creates a `SERIAL` column, or `BIGSERIAL` for
type `bigint`. `defaultValue: { special: 'CURRENT_TIMESTAMP' }` sets the
default to `CURRENT_TIMESTAMP`.

## changeColumn

`changeColumn` applies the complete spec given:

- `notNull: true` sets `NOT NULL`, otherwise it is dropped.
- `unique: true` adds the constraint `<table>_<column>_key`, `unique: false`
  drops it, without `unique` nothing changes.
- `defaultValue` sets the default, without it the default is dropped.
- `type` (and `length`) changes the type, converting with
  `USING "<column>"::<type>`. Set `using` to give your own `USING` clause.

```js
db.changeColumn('pets', 'age', {
  type: 'int',
  notNull: true,
  defaultValue: 0,
  using: 'USING "age"::integer'
});
```

## Further notes

- `removeColumn`, `renameColumn`, `changeColumn`, `renameTable` and
  `removeForeignKey` accept an options object in place of the callback, used
  for the [column strategies](../Guides/migrations v2.md#removing-notnull-columns)
  of v2 migrations.
- `runSql` takes `?` placeholders, they are translated to `$1`, `$2`, ...
  `all` passes the query on unchanged, use `$1`, `$2`, ... there.
- `db:create` fails if the database exists already, `db:drop` uses
  `IF EXISTS`.
- v1 migrations run inside `BEGIN` ... `COMMIT`, unless
  `--non-transactional`.
- Errors show the position of the failed statement reported by PostgreSQL,
  see [Failures and recovery](../Guides/failures and recovery.md).
- Supports the [migration lock](../Guides/running in parallel.md) since 1.6.0.
- Since 1.6.1, a connection error is reported once, and an unsupported special
  default value is dropped with a warning instead of failing.
