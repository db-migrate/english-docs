# Data migrations

A data migration changes the data instead of the schema. It is a
[v2 migration](migrations v2.md) of the type `dml`, and like a schema
migration it needs no down function: db-migrate records how to revert every
instruction and reverts it on its own.

```js
exports.migrate = async (db, opt) => {
  await db.insert('kinds', [{ name: 'dog' }, { name: 'cat' }]);
  await db.update('pets', { kind: 'dog' }, { kind: 'hound' });
  await db.delete('pets', { name: null });
};

exports._meta = {
  version: 2,
  type: 'dml'
};
```

Data migrations need db-migrate 1.3.0 and db-migrate-base 2.5.0, which comes
with db-migrate-pg 1.7.0, db-migrate-mysql 3.2.0, db-migrate-sqlite3 1.2.0 and
db-migrate-cockroachdb 5.9.0.

## Instructions

Every instruction is a step of the migration, like the instructions of a
schema migration.

| Instruction | Reverted by |
|---|---|
| `insert(table, rows, [options])` | deleting the inserted rows |
| `update(table, set, where, [options])` | restoring the previous values |
| `delete(table, where, [options])` | inserting the deleted rows again |
| `runSql(sql, [params], options)` | the SQL given with `revert` |

`all(sql, [params])` reads rows, it is no step and changes nothing.

Schema instructions like `createTable` fail in a data migration, change the
schema in a schema migration of its own.

### insert

`insert` takes the rows in any form of the [SQL API](../API/SQL.md#inserttablename-rows-callback):
an object, an array of objects, `{ columns, data }`, or the columns and the
values.

Every inserted row gets the migration and step in its `__dbmigrate__flag`
column, the column v2 migrations add to every table they create. Reverting
deletes the rows with that flag, so rows inserted by others are never touched.
Tables created by v1 migrations, or with `noDefaultColumn`, have no such
column, `insert` fails for them unless `{ irreversible: true }` is passed.

### update and delete

`where` selects the rows:

- an object, the columns equal to the values, `null` for `IS NULL` and arrays
  for `IN`: `{ kind: 'dog', owner_id: [1, 2], name: null }`
- SQL: `"kind = 'dog'"`
- SQL with parameters: `['kind = ? AND age > ?', ['dog', 3]]`

```js
await db.update('pets', { kind: 'dog', name: 'unknown' }, { kind: 'hound', name: null });
await db.delete('pets', ['born < ?', ['2000-01-01']]);
```

Before changing anything, the rows are copied to a backup table in the same
database, `__dbm_backup_<hash>`. `update` copies the key and the columns it
changes, `delete` the whole rows. The rows are then changed by their key, in
batches of 1000, set another size with `{ batch: 5000 }`. Reverting restores
them from the backup and drops it.

The key is the primary key of the table, as created by v2 migrations. For
other tables pass it, `{ key: 'id' }` or `{ key: ['a', 'b'] }`. An update can
not change the key itself.

The backup tables stay until the migration is reverted, as long as it may be
reverted. Rows deleted by the database on its own, e.g. by a foreign key with
`ON DELETE CASCADE`, are not in the backup, delete them explicitly first if
reverting has to bring them back.

### runSql

Raw SQL needs the SQL reverting it, or `irreversible`:

```js
await db.runSql('UPDATE pets SET age = age + 1', {
  revert: 'UPDATE pets SET age = age - 1'
});
await db.runSql('UPDATE pets SET kind = ? WHERE kind = ?', ['dog', 'hound'], {
  revert: ['UPDATE pets SET kind = ? WHERE kind = ?', ['hound', 'dog']]
});
```

## Irreversible steps

`{ irreversible: true }` runs an instruction without recording how to revert
it, without flag or backup. A migration with an irreversible step can not be
reverted, `down` refuses it. If such a migration fails, it is not rolled back,
the steps executed stay and the next run continues after them, see
[Failures and recovery](failures and recovery.md).

## Large tables

An update or delete of many rows runs in batches, the progress is kept in the
state after every batch. If the process dies in between, the next run
continues the step with the next batch, using the backup taken before, see
[Failures and recovery](failures and recovery.md). With `_meta.recovery:
'rollback'`, the executed steps are reverted and the migration runs again
instead.

## Dry run

With `--dry-run`, the statements of the instructions are printed, `update` and
`delete` as a single statement on `where`, without backup.
