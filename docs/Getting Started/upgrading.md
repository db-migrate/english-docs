# Upgrading from 0.11

db-migrate 0.11 and older are end of life. Your existing migrations keep
working with 1.0 unchanged, so for most projects upgrading means installing
1.0 and checking the breaking changes below.

    $ npm install db-migrate@1

## Breaking changes

### Node.js 24

Node.js 24 and newer are supported officially. db-migrate does not block older
versions from installing and they may keep working, but they are not tested.

### SSH tunnels need a plugin

The ssh tunnel is no longer built in. If your config has a `tunnel` section,
install the plugin next to db-migrate, your config stays the same:

    $ npm install db-migrate-plugin-tunnel-ssh

See [Plugins](plugins.md).

### Seeders are dropped

The seeders were never finished. In 1.0, `db-migrate seed` and the seed
functions of the programmable API fail with a clear message instead of
silently doing nothing. A new concept for seeding follows separately.

### transition is removed

The `transition` command converted migrations of db-migrate before 0.9. Use
0.11 to transition such migrations first, then upgrade.

### The state table

Next to the migrations table, db-migrate creates and maintains a state table,
`migrations_state` by default. It holds the migration lock and the progress of
running migrations. Its name can be set with `--state-table`. Keep it, deleting
it loses the lock and the information needed to recover interrupted runs.

## Update your driver

Install the current version of your driver to get the
[migration lock](../Guides/running in parallel.md):

| Driver | Version |
|---|---|
| db-migrate-pg | 1.6.0 |
| db-migrate-mysql | 3.1.0 |
| db-migrate-sqlite3 | 1.1.0 |
| db-migrate-cockroachdb | 5.8.1 |

Older drivers keep working, db-migrate then warns that migrations run without a
lock.

## What you get

- A [migration lock](../Guides/running in parallel.md), so several processes
  can start migrating the same database at once.
- [Migrations without a down function](../Guides/migrations v2.md), reverted
  and recovered by db-migrate on its own.
- [Error output](../Guides/failures and recovery.md) naming the failed
  migration, instruction and statement.
- [Plain SQL migrations](plugins.md) with db-migrate-plugin-sql.
