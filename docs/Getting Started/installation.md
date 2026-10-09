# Installation

Officially supported is Node.js 24 and newer. Older versions may work, but are
not tested.

Install db-migrate and the driver of your database in your project:

    $ npm install db-migrate db-migrate-pg

The official drivers are `db-migrate-pg`, `db-migrate-mysql`,
`db-migrate-sqlite3`, `db-migrate-cockroachdb` and `db-migrate-mongodb`, see
[Drivers](../drivers.md).

Run it through npm:

    $ npx db-migrate up

or from the scripts of your `package.json`:

```json
{
  "scripts": {
    "migrate": "db-migrate up"
  }
}
```

db-migrate can be installed globally as well:

    $ npm install -g db-migrate

The global `db-migrate` always runs the version installed in your project, if
there is one, and falls back to the global version otherwise.

Run db-migrate from the root of your project: it reads the `package.json`
there to find its [plugins](plugins.md), and resolves `database.json` and the
`migrations` directory relative to the current directory by default.

Next: [Configuration](configuration.md), [Usage](usage.md) and the
[Commands](commands.md).
