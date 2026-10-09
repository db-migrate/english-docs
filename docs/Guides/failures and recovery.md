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

v1 migrations are not rolled back by db-migrate. Depending on the driver, they
run inside a transaction.

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
