# MongoDB

    $ npm install db-migrate-mongodb

Based on the [MongoDB Node.js driver](https://github.com/mongodb/node-mongodb-native).
Migrations use the [NoSQL API](../API/NoSQL.md).

## Configuration

```json
{
  "dev": {
    "driver": "mongodb",
    "host": "localhost",
    "database": "app"
  },
  "prod": {
    "driver": "mongodb",
    "host": ["mongo1.example.com:27017", "mongo2.example.com:27017"],
    "database": "app",
    "user": "app",
    "password": {"ENV": "MONGO_PASSWORD"},
    "authSource": "admin",
    "replicaSet": "rs0",
    "ssl": true
  }
}
```

| Setting | |
|---|---|
| `database` | required |
| `host` | a host, or a list of hosts as `"host:port"` strings or `{ "host", "port" }` objects. Default `localhost`. `hosts` works as well. |
| `port` | default `27017`, for hosts without a port |
| `user`, `password` | credentials, special characters are encoded for you |
| `authSource` | the authentication database, used with `user` and `password` |
| `replicaSet` | name of the replica set |
| `ssl` | `true` to connect with ssl |

## Limitations

The MongoDB driver does not implement the state management of db-migrate 1.0.
db-migrate warns about it on every run and works as before 1.0:

- no [migration lock](../Guides/running in parallel.md), do not run several
  processes migrating at once
- no [v2 migrations](../Guides/migrations v2.md), only v1 migrations
- no transactions
- a [scope](../Getting Started/commands.md#scope-configuration) `config.json`
  only switching the `database` has no effect
- `db:create` and `db:drop` are not supported
