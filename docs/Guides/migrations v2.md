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

`migrate` receives the instructions as `db` and an options object:

| `opt` | |
|---|---|
| `dbm` | `{ version, dataType }`, db-migrate and its [data types](../API/generic datatypes.md) |
| `options` | the same options `setup` of a v1 migration receives, see [Usage](../Getting Started/usage.md#creating-migrations) |

Every instruction returns a promise, there are no callbacks.

`_meta` takes:

| `_meta` | |
|---|---|
| `version` | `2`, required |
| `noDefaultColumn` | `true` to not add the [default column](#the-default-column) |
| `recovery` | `'skip'` (default) or `'rollback'`, see [Failures and recovery](failures and recovery.md#interrupted-runs-v2) |

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
`changePrimaryKey`, see [CockroachDB](../Drivers/cockroachdb.md#additional-instructions).

Raw SQL (`runSql`), `insert` and `all` are not available in v2 migrations, as
db-migrate can not learn what they do. Use a v1 migration for them.

### Removing notNull columns

Removing a column with `notNull: true` needs a strategy, so it can be brought
back without failing on the existing rows:

- `{ columnStrategy: 'defaultValue', passthrough: { defaultValue: 'x' } }`
  restores the column with a default value.
- `{ columnStrategy: 'delay' }` renames the column instead of dropping it, so
  it can be renamed back.

```js
await db.removeColumn('pets', 'name', {
  columnStrategy: 'defaultValue',
  passthrough: { defaultValue: '' }
});
```

The driver has to support column strategies, db-migrate-pg and
db-migrate-cockroachdb do.

## Objects created outside of v2 migrations

db-migrate only knows the schema its v2 migrations taught it. A table created
by a v1 migration, with `runSql` or by hand is unknown to it, and v2
instructions on it fail:

```
The table "legacy" is unknown to the schema of db-migrate, it was not created by a v2 migration. Declare it first with db.adopt.createTable("legacy", columns), or pass { irreversible: true } to drop it without being able to revert it.
```

### Adopting

`db.adopt` declares an existing object to the schema, without executing
anything on the database and without adding the default column. Afterwards
v2 migrations can change or drop it like their own objects, fully revertible:

```js
exports.migrate = async (db) => {
  // legacy was created by a v1 migration
  await db.adopt.createTable('legacy', {
    id: { type: 'int', primaryKey: true },
    name: { type: 'string', length: 20 }
  });

  await db.addColumn('legacy', 'age', { type: 'int' });
  await db.removeColumn('legacy', 'name');
};

exports._meta = {
  version: 2
};
```

Reverting this migration removes `age` and adds `name` back from its adopted
definition. Reverting the adopt itself only forgets the table again, it stays
in the database.

`db.adopt` offers `createTable`, `addColumn`, `addIndex`, `addForeignKey` and
every other instruction starting with `create` or `add`, including those of
the driver, like `createEnum` of db-migrate-cockroachdb. Describe the object
as it exists, adopting an object already known to the schema fails.

### Dropping irreversibly

To drop an unknown object without declaring it, pass
`{ irreversible: true }`:

```js
await db.dropTable('legacy', { irreversible: true });
await db.removeColumn('pets', 'legacy_flag', { irreversible: true });
await db.removeForeignKey('pets', 'pets_legacy_fk', { irreversible: true });
```

Such a step can not be reverted, and with it the whole migration:

- If the migration fails, it is not rolled back. The executed steps stay, the
  next run continues after them.
- `db-migrate down` refuses to revert it, before changing anything:

```
Migration "20260101000003-drop" can not be reverted, it ran step 1 dropTable("junk") with { irreversible: true }.
```

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

## Rebuilding the learned schema

The learned schema is stored in the state table. If it got lost or out of
sync, `db-migrate fix` rebuilds it from the executed v2 migrations without
running anything on the database, see [Commands](../Getting Started/commands.md#fix).

## Driver support

v2 migrations need a driver with the state management of db-migrate 1.0:
db-migrate-pg, db-migrate-mysql, db-migrate-sqlite3 and db-migrate-cockroachdb.
A v2 migration can only use instructions its driver implements, and reverting
needs the reverse instruction as well, e.g. sqlite3 can not remove columns,
so it can not revert `addColumn`. MongoDB does not support v2 migrations. See
[Drivers](../drivers.md).

## Limitations

- The data of a dropped table or column can not be restored by a rollback.
- Steps run with `{ irreversible: true }` can not be reverted.
