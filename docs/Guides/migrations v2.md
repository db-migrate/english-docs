# Migration schema v2

A v2 migration has no down function. db-migrate learns the schema from the
instructions you run and stores how to revert each of them, so it can undo the
migration on `db-migrate down`, roll it back on its own when it fails, and
recover it when a run was interrupted.

v1 migrations (`exports.up` and `exports.down`) keep working unchanged, both
kinds can be mixed in one migrations directory.

## A v2 migration

    $ db-migrate create add-pets --v2-file

```js
'use strict';

exports.migrate = async (db, opt) => {
  const type = opt.dbm.dataType;

  await db.createTable('pets', {
    id: { type: type.INTEGER, primaryKey: true },
    name: { type: type.STRING, length: 48, notNull: true }
  });

  await db.addIndex('pets', 'pets_name_idx', ['name']);
};

exports._meta = {
  version: 2
};
```

`migrate` receives the driver and an options object, `opt.dbm` gives access to
db-migrate, for example to its data types. Every instruction returns a promise,
there are no callbacks.

## Instructions

A v2 migration can only run instructions db-migrate knows how to revert:

| Instruction | Reverted by |
|---|---|
| `createTable(table, columns)` | dropping the table |
| `dropTable(table)` | recreating it with its columns, indexes and foreign keys, but **not its data** |
| `renameTable(table, newName)` | renaming it back |
| `addColumn(table, column, spec)` | removing the column |
| `removeColumn(table, column, [options])` | adding it back, see below |
| `renameColumn(table, column, newName)` | renaming it back |
| `changeColumn(table, column, spec)` | restoring the previous spec |
| `addIndex(table, index, columns, [unique])` | removing the index |
| `removeIndex(table, index)` | adding it back |
| `addForeignKey(table, refTable, key, mapping, rules)` | removing the foreign key |
| `removeForeignKey(table, key)` | adding it back |

Drivers can add further instructions. db-migrate-cockroachdb adds
`createEnum`, `dropEnum`, `renameEnum`, `addEnumType`, `dropEnumType` and
`changePrimaryKey`.

Raw SQL (`runSql`) is not available in v2 migrations, as db-migrate can not
learn what it does. Use a v1 migration for it.

### Removing notNull columns

Removing a column with `notNull: true` needs a strategy, so it can be brought
back without failing on the existing rows:

- `{ columnStrategy: 'defaultValue', passthrough: { defaultValue: 'x' } }`
  restores the column with a default value.
- `{ columnStrategy: 'delay' }` renames the column instead of dropping it, so
  it can be renamed back.

The driver has to support column strategies, db-migrate-pg and
db-migrate-cockroachdb do.

## The default column

db-migrate adds a column `__dbmigrate__flag` to every table created by a v2
migration. Set `noDefaultColumn` to leave it out:

```js
exports._meta = {
  version: 2,
  noDefaultColumn: true
};
```

## Failures and interruptions

A v2 migration runs outside of a transaction. Every step is recorded in the
state table before it runs, which is what makes the automatic rollback and the
recovery of interrupted runs possible. See
[Failures and recovery](failures and recovery.md).

## Limitations

- db-migrate only knows the schema its v2 migrations taught it. A table
  created by a v1 migration, or by hand, is unknown to it, and v2 instructions
  on it fail with `There is no ... table in schema!`.
- The data of a dropped table or column can not be restored by a rollback.
