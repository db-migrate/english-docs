## Development

The following command runs the vows tests.

```bash
npm test
```

If tests fail because of timeout you can add TIMEOUT parameter.
```bash
npm test -e "TIMEOUT=5000"
```

Running the tests requires a one-time setup of the **MySQL**, **MongoDB** and **Postgres** databases.

```bash
mysql -u root -e "CREATE DATABASE db_migrate_test;"
createdb db_migrate_test
```

You will also need to copy `test/db.config.example.json` to `test/db.config.json`
and adjust appropriate to setup configuration for your database instances.
