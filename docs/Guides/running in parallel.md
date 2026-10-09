# Running in parallel

Deployments often start several instances of an application at once, each of
them running `db-migrate up` before starting. db-migrate coordinates them
through a lock in the [state table](../Getting Started/upgrading.md#the-state-table),
so only one of them migrates.

## How it works

1. Every process first determines the pending migrations. If there are none,
   it is done, without ever touching the lock.
2. If there are pending migrations, the process tries to acquire the lock.
3. The process holding the lock runs the migrations and releases the lock
   afterwards.
4. Every other process waits until the lock is released, then determines the
   pending migrations again. Usually the other process ran them already, and
   there is nothing left to do.

The lock is acquired by an atomic update of the state table, so exactly one
process wins, whatever the timing.

## Processes that die

While it migrates, the process holding the lock keeps updating it. If a waiting
process sees no change of the lock for `--lock-timeout` milliseconds, it
considers the holder dead and takes the lock over. The waiting processes
measure this on their own clock, so the clocks of the machines do not need to
be in sync.

If the dead process was in the middle of a v2 migration, the next run recovers
it, see [Failures and recovery](failures and recovery.md).

Keep `--lock-timeout` well above the time a single statement of your migrations
may block the process holding the lock, otherwise a busy process could be
taken for dead.

## Options

| Option | Default | Description |
|---|---|---|
| `--lock-timeout` | `60000` | Milliseconds without any sign of life after which a lock is taken over. |
| `--lock-interval` | `1000` | Milliseconds between checks while waiting for the lock. |

Both can be set in the [rc config](../Getting Started/configuration.md#rc-configs)
as well:

```json
{
  "lock-timeout": 120000
}
```

## Driver support

The lock needs a driver declaring support for it:

| Driver | Since |
|---|---|
| db-migrate-pg | 1.6.0 |
| db-migrate-mysql | 3.1.0 |
| db-migrate-sqlite3 | 1.1.0 |
| db-migrate-cockroachdb | 5.8.0 |

With any other driver db-migrate warns and migrates without a lock, as before
1.0. Drivers declare their support with `_meta.supports.locking`.
