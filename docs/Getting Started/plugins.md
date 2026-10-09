# Plugins

Plugins extend db-migrate. Install them next to db-migrate in your project,
db-migrate loads every dependency of your `package.json` whose name starts with
`db-migrate-plugin`. There is nothing to configure.

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

SQL and JavaScript migrations can be mixed in one migrations directory. The
built-in `--sql-file` mode, a JavaScript migration loading `sqls/*-up.sql` and
`sqls/*-down.sql`, keeps working as before.

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
host. db-migrate connects through `localhost:localPort`. Instead of
`privateKeyPath`, `password` or `privateKey` work as well, further options are
passed on to [ssh2](https://github.com/mscdex/ssh2#client-methods). The tunnel
closes once db-migrate is done, set `keepAlive: true` to keep it open.

## YAML config

[db-migrate-plugin-yaml](https://github.com/db-migrate/plugin-yaml) reads the
database config from a YAML file.

## Writing plugins

See [Writing plugins](../Developers/writing plugins.md).
