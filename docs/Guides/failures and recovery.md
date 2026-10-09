# Failures and recovery

## Error output

When a migration fails, db-migrate tells you what failed and exits with code 1:

```
[ERROR] Migration "20261009000002-pets" failed at step 2 addColumn("pets", "x"): type "notatype" does not exist
    SQL: ALTER TABLE "pets" ADD COLUMN "x" NOTATYPE
                                           ^
    code: 42704
    error: type "notatype" does not exist
        at Parser.parseErrorMessage (...)
```

- the migration, and for v2 migrations the instruction and its step. "after
  step" means the instruction succeeded and the code of the migration failed
  afterwards.
- the failed statement, with a marker at the position reported by the database
  (PostgreSQL, CockroachDB). Statements with parameters show the position only.
- the diagnostic fields of the driver, like `code`, `detail`, `hint`,
  `constraint` or `errno`.
- always the stack.

Run with `--verbose` to see the complete error of the driver as well.

The programmable API rejects with the same error, carrying the properties
`migration`, `instruction` and `sql`.

## Automatic rollback (v2)

When a v2 migration fails, db-migrate rolls it back: it reverts exactly the
steps that reached the database. A step that failed on the database is not
reverted, unless its main statement went through before failing (e.g. a table
was created, but adding its foreign key failed).

A migration that ran a step with `{ irreversible: true }` is not rolled back,
as that step can not be reverted:

```
[ERROR] Migration "20261009000003-legacy" failed and can not be rolled back, it ran a step with { irreversible: true }. The steps executed stay, the next run continues after them.
```

The next `db-migrate up` [recovers](#interrupted-runs-v2) it like an
interrupted run. See
[Dropping irreversibly](migrations v2.md#dropping-irreversibly).

v1 migrations are not rolled back by db-migrate. The SQL drivers run them
inside a transaction (unless `--non-transactional`), so whatever the database
can roll back is undone. See the [driver pages](../drivers.md).

## Interrupted runs (v2)

A process may die in the middle of a v2 migration: killed, crashed or cut off
from the database. Or the rollback after a failure fails itself. Then the next
`db-migrate up` finds the migration unfinished in the state table and recovers
it before going on, logging how:

```
[WARN] [recovery] 20261009000001-pets: the previous run was interrupted at step 2, 2 steps were executed, recovering by skip
[INFO] [recovery] 20261009000001-pets: skipping already executed step 1/2 createTable("pets")
[INFO] [recovery] 20261009000001-pets: skipping already executed step 2/2 addColumn("pets", "x")
```

How is set per migration:

```js
exports._meta = {
  version: 2,
  recovery: 'skip'
};
```

| `recovery` | |
|---|---|
| `skip` (default) | Skip the steps already executed and continue with the rest. |
| `rollback` | Revert the steps already executed, then run the migration again. |

- A run interrupted while rolling back always continues the rollback.
- If the migration file changed since the interruption, `skip` is refused, as
  the steps might not match anymore. Set `recovery: 'rollback'`, or repair the
  database by hand.
- A step that was sent to the database but not confirmed before the process
  died is run again with `skip`. If it went through, it fails with "already
  exists" and has to be repaired by hand. The window for this is very small.

## Data migrations

[Data migrations](data migrations.md) are rolled back and recovered the same
way, with one difference: the step a data migration failed or was interrupted
at is always safe to revert or to run again.

- A failed step is reverted with the others, its rows inserted so far are
  deleted, its rows changed so far are restored from the backup.
- An interrupted `insert` deletes the rows of its previous attempt and inserts
  them again.
- An interrupted `update` or `delete` continues with the batch after the last
  one recorded, using the backup taken by its previous attempt:

```
[INFO] [recovery] 20261009000002-pet-data: continuing interrupted step 1 update("pets", {"kind":"hound"})
```

- An interrupted revert is continued by the next `db-migrate down`.
- A `runSql` interrupted before it was confirmed is run again, like a step of
  a schema migration.

Since db-migrate 1.4.0 each step, and each batch of `update` and `delete`,
runs in a transaction, so nothing of it is left behind half done. The jobs of
[background migrations](background migrations.md) are recovered the same way
by the worker taking them over.
