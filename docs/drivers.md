# Drivers

db-migrate talks to databases through drivers, installed next to db-migrate
and selected by the `driver` setting of an
[environment](Getting Started/configuration.md).

| Database | Package | `driver` | Page |
|---|---|---|---|
| PostgreSQL | `db-migrate-pg` | `pg`, `postgres`, `postgresql` | [PostgreSQL](Drivers/pg.md) |
| MySQL, MariaDB | `db-migrate-mysql` | `mysql` | [MySQL](Drivers/mysql.md) |
| SQLite | `db-migrate-sqlite3` | `sqlite3`, `sqlite` | [sqlite3](Drivers/sqlite3.md) |
| CockroachDB | `db-migrate-cockroachdb` | `cockroachdb` | [CockroachDB](Drivers/cockroachdb.md) |
| MongoDB | `db-migrate-mongodb` | `mongodb` | [MongoDB](Drivers/mongodb.md) |

Any other driver is loaded as `db-migrate-<driver>`, or from a path with
`"driver": { "require": "./my-driver" }`.

## Features

| | pg | mysql | sqlite3 | cockroachdb | mongodb |
|---|---|---|---|---|---|
| [Migration lock](Guides/running in parallel.md) | since 1.6.0 | since 3.1.0 | since 1.1.0 | since 5.8.0 | no |
| [v2 migrations](Guides/migrations v2.md) | yes | yes | yes, limited to what the driver implements | yes | no |
| Column strategies (v2) | yes | no | no | yes | no |
| Transactions for v1 migrations | yes | yes | yes | yes | no |
| Scope `config.json` | sets the `search_path` | switches the database | ignored | sets the `search_path` | no effect |

The SQL drivers share the [SQL API](API/SQL.md), the driver pages list their
additions and differences. MongoDB has its own [NoSQL API](API/NoSQL.md).

Older drivers keep working with db-migrate 1.0, db-migrate warns that they run
without the migration lock, or without its state management at all.
