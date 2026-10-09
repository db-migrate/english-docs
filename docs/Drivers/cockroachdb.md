# CockroachDB

    $ npm install db-migrate-cockroachdb

Built on the [PostgreSQL driver](pg.md).

## Configuration

```json
{
  "dev": {
    "driver": "cockroachdb",
    "host": "localhost",
    "port": 26257,
    "user": "root",
    "database": "app"
  }
}
```

The settings are passed to `pg.Client` like with the
[PostgreSQL driver](pg.md#configuration), including `ssl` with `sslmode`,
`sslrootcert`, `sslcert` and `sslkey`, and `native`.

## Data types

In addition to the types of the [PostgreSQL driver](pg.md#data-types):
`uuid`, `jsonb`, `float`, `timestamptz`, `enum` and `computed`.

| Type | |
|---|---|
| `enum` | an ENUM created with `createEnum`, named by `enumName` |
| `computed` | a computed column: `computedType` is its type, `function` the expression, `stored: true` stores it |

```js
db.createTable('pets', {
  id: { type: 'uuid', primaryKey: true, autoIncrement: true },
  kind: { type: 'enum', enumName: 'pet_kind' },
  name: 'string',
  name_upper: {
    type: 'computed',
    computedType: 'string',
    function: 'upper(name)',
    stored: true
  }
});
```

## Column spec

In addition to the [common column spec](../API/SQL.md#column-spec):

| Option | |
|---|---|
| `autoIncrement` | on a primary key: `uuid` gets the default `gen_random_uuid()`, other types become `SERIAL` |
| `onUpdate` | `ON UPDATE` expression, like `defaultValue` |
| `family` | name of the column family |
| `interleave` | table to interleave a primary key column in, deprecated by CockroachDB |
| `comment` | adds a comment to the column |

`defaultValue` and `onUpdate` take `{ special: 'NOW' }` for `NOW()` and
`{ special: 'CURRENT_TIMESTAMP' }` for `CURRENT_TIMESTAMP()`, or
`{ raw: '...' }` for any expression.

Columns with a `foreignKey` get an index named like the foreign key.

## Table options

`createTable` with a `columns` object takes row level TTL settings:

| Option | |
|---|---|
| `expire` | column for `ttl_expiration_expression` |
| `expireAfter` | `ttl_expire_after`, e.g. `'30 days'` |
| `ttlJobCron` | `ttl_job_cron` |

## Indexes

`addIndex(table, name, columns, options)` takes `true` or
`{ unique, inverted }` as options. Columns can be names or
`{ name, DESC, opclass }`, `DESC: true` for descending and `false` for
ascending order:

```js
db.addIndex('pets', 'pets_tags', [{ name: 'tags' }], { inverted: true });
```

`removeIndex(table, index)` drops `table@index`.

## Additional instructions

Available in v1 and [v2 migrations](../Guides/migrations v2.md), where they are
reverted automatically:

| Instruction | |
|---|---|
| `changePrimaryKey(table, columns)` | replace the primary key by the given columns |
| `createEnum(name, values)` | `CREATE TYPE ... AS ENUM` |
| `dropEnum(name)` | drop an ENUM |
| `renameEnum(name, newName)` | rename an ENUM |
| `addEnumType(name, value)` | add a value to an ENUM |
| `dropEnumType(name, value)` | drop a value of an ENUM |

```js
exports.migrate = async (db) => {
  await db.createEnum('pet_kind', ['cat', 'dog']);
  await db.addEnumType('pet_kind', 'bird');
};
```

In a v2 migration, dropping an ENUM unknown to the learned schema warns that
it can not be reverted.

## changeColumn

Unlike the PostgreSQL driver, `notNull` is only changed if given. `unique`
adds or drops the constraint `<table>_<column>_key`. Without `defaultValue`
the default is dropped, without `onUpdate` the `ON UPDATE` expression. `type`
is passed on as is.

## Further notes

- v1 migrations run inside `BEGIN` ... `COMMIT`, unless `--non-transactional`.
- Supports [column strategies](../Guides/migrations v2.md#removing-notnull-columns).
- Supports the [migration lock](../Guides/running in parallel.md) since 5.8.0.
