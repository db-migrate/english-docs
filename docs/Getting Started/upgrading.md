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
silently doing nothing. Since 1.3.0 there are [seeds](../Guides/seeds.md) and
[data migrations](../Guides/data migrations.md) instead. The options
`--seeds-table`, `--vcseeder-dir` and `--staticseeder-dir` are still accepted,
but have no effect.

### transition is removed

The `transition` command converted migrations of db-migrate before 0.9. Use
0.11 to transition such migrations first, then upgrade.

### The state table

Next to the migrations table, db-migrate creates and maintains a state table,
`migrations_state` by default. It holds the migration lock and the progress of
running migrations. Its name can be set with `--state-table`. Keep it, deleting
it loses the lock and the information needed to recover interrupted runs.

## Upgrading from 1.0 to 1.1

- A scope whose `config.json` switches the `database` or `schema` keeps its
  migration lock, recovery progress and learned schema in that database or
  schema now. If v2 migrations of such a scope ran with 1.0, run
  `db-migrate fix:<scope>` once to learn their schema there. See
  [Scope configuration](commands.md#scope-configuration).
- With PostgreSQL, update to db-migrate-pg 1.6.1, 1.6.0 ignores the `schema`
  setting, see [PostgreSQL](../Drivers/pg.md#schema).

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
db-migrate-mongodb has no lock and no state management, see
[Drivers](../drivers.md).

## What you get

- A [migration lock](../Guides/running in parallel.md), so several processes
  can start migrating the same database at once.
- [Migrations without a down function](../Guides/migrations v2.md), reverted
  and recovered by db-migrate on its own.
- [Error output](../Guides/failures and recovery.md) naming the failed
  migration, instruction and statement.
- [Plain SQL migrations](plugins.md) with db-migrate-plugin-sql.
