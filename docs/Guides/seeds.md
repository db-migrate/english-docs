# Seeds

Seeds insert data for development and tests, like example users or the
fixtures of your tests. Unlike [data migrations](data migrations.md), they
are not part of the history of a database: a seed can be changed and run again
any time, it replaces the rows it inserted before.

Seeds need db-migrate 1.3.0 and db-migrate-base 2.5.0, which comes with
db-migrate-pg 1.7.0, db-migrate-mysql 3.2.0, db-migrate-sqlite3 1.2.0 and
db-migrate-cockroachdb 5.9.0.

## Writing seeds

A seed is a file in the `seeds` directory, `seeds/<name>.js`, exporting a
`seed` function:

```js
exports.seed = async (db, opt) => {
  await db.insert('owners', [
    { id: 1, name: 'Ann' },
    { id: 2, name: 'Bob' }
  ]);

  const owners = await db.all('SELECT id FROM owners');
  await db.insert('pets', owners.map(o => ({ owner_id: o.id, name: 'Rex' })));
};
```

Seeds can only `insert`, in any form of the
[SQL API](../API/SQL.md#inserttablename-rows-callback), and read with `all`.
They insert into tables created by [v2 migrations](migrations v2.md) only:
every row gets `seed:<name>` in its `__dbmigrate__flag` column, by which it is
removed again.

The seeds run in the order of their file names, prefix them with numbers to
seed tables referenced by others first, e.g. `1-owners.js` and `2-pets.js`.
Set another directory with `--seeds-dir`.

## Running seeds

    $ db-migrate seed

removes the rows of all seeds run before, the last first, and runs all seeds
again. Rows inserted by others stay untouched. A seed whose file was removed
has its rows removed as well.

    $ db-migrate seed pets

runs only the seed `pets`, replacing its rows.

    $ db-migrate seed down pets
    $ db-migrate seed reset

remove the rows of the seed `pets`, or of all seeds, without running them.

Removing the rows of a single seed fails, if rows of other seeds still
reference them by a foreign key. Run `db-migrate seed` without a name to seed
everything again in the right order.

If a seed fails, its rows inserted so far stay until the next run removes
them.
