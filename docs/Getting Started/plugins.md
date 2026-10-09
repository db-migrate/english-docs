# Plugins

Plugins extend db-migrate. Install them next to db-migrate in your project,
db-migrate loads every entry of `dependencies` and `devDependencies` in the
`package.json` of the current directory whose name starts with
`db-migrate-plugin`, from `node_modules`. There is nothing to configure.

With the [programmable API](../API/programable.md), `noPlugins: true` skips
loading them and `plugins` registers plugins directly.

## Plain SQL migrations

[db-migrate-plugin-sql](https://github.com/db-migrate/plugin-sql) runs
migrations written as plain SQL files, without any JavaScript.

    $ npm install db-migrate-plugin-sql
    $ db-migrate create add-users --sql

This creates `migrations/<timestamp>-add-users.sql`, with an up and an optional
down section:

```sql
-- up
CREATE TABLE users (id int PRIMARY KEY, name text);

-- down
DROP TABLE users;
```

Each section starts with its own line `-- up` or `-- down`, only comments may
come before the first one. The SQL of a section is run with one `runSql`, an
empty section does nothing. Whether a section may contain several statements
depends on the driver, for MySQL enable
[multipleStatements](../Drivers/mysql.md#multiplestatements).

SQL migrations are v1 migrations, they run inside a transaction like other v1
migrations. SQL and JavaScript migrations can be mixed in one migrations
directory. The built-in `--sql-file` mode, a JavaScript migration loading
`sqls/*-up.sql` and `sqls/*-down.sql`, keeps working as before.

## SSH tunnels

[db-migrate-plugin-tunnel-ssh](https://github.com/db-migrate/plugin-tunnel-ssh)
connects to the database through an ssh tunnel.

    $ npm install db-migrate-plugin-tunnel-ssh

Configure the tunnel in your environment:

```json
{
  "prod": {
    "driver": "pg",
    "host": "db.internal",
    "port": 5432,
    "database": "app",
    "tunnel": {
      "host": "bastion.example.com",
      "port": 22,
      "username": "deploy",
      "privateKeyPath": "/home/deploy/.ssh/id_ed25519",
      "localPort": 33333
    }
  }
}
```

`host` and `port` of the environment are the database as seen from the ssh
host. db-migrate connects through `127.0.0.1:localPort`, so `localPort` is
required. Instead of `privateKeyPath`, `password` or `privateKey` work as
well, further options are passed on to
[ssh2](https://github.com/mscdex/ssh2#client-methods). The tunnel closes once
db-migrate is done, set `keepAlive: true` to keep it open.

Without the plugin, an environment with a `tunnel` fails with
`A ssh tunnel is configured, but no plugin provides it`.

## YAML config

[db-migrate-plugin-yaml](https://github.com/db-migrate/plugin-yaml) reads the
config file as YAML:

    $ npm install db-migrate-plugin-yaml
    $ db-migrate up --config database.yml

```yaml
dev:
  driver: pg
  database: app
  user:
    ENV: PGUSER
```

If the file can not be read as YAML, db-migrate warns and falls back to
reading it on its own.

## Writing plugins

See [Writing plugins](../Developers/writing plugins.md).
