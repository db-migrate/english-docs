# Releases and deprecation

Removing a table or column right away breaks the version of your application
still running during a deployment, and every older version you might roll
back to. db-migrate removes them in stages instead, along the releases of your
application.

Releases need db-migrate 1.5.0 and [v2 migrations](migrations v2.md).

## Releases

A migration starts a release with `_meta.release`:

```js
exports._meta = {
  version: 2,
  release: '2.3.0'
};
```

The label can be anything, a version, a date or a name. db-migrate does not
interpret it, it only counts the releases in the order of the migrations: a
migration without `release` belongs to the release before it. Two releases
after `2.3.0` are the second and third label following it, whatever they are.

## Deprecating

```js
exports.migrate = async db => {
  await db.deprecateTable('legacy_orders');
  await db.deprecateColumn('users', 'fax');
};
```

marks the table and column in the release of the migration, call it R. Your
application stops using them in R. A column with `notNull` is relaxed, so the
application can stop writing it (not with sqlite, which can not change
columns).

| | |
|---|---|
| R | marked, `notNull` of a column relaxed |
| R+1 | renamed to `__dbm_deprecated_<name>_<time>`, with the time of the migration deprecating it, whatever still uses it fails now, while the data is still there |
| R+N | dropped, with `drop: 'auto'` only |

The renaming and dropping happen before the first migration of the release,
logged as:

```
[INFO] [release] starting release 2.4.0
[INFO] [release] renaming the deprecated table "legacy_orders" to __dbm_deprecated_legacy_orders_20261009120000
```

## Dropping

`N` is 4 by default. With `drop: 'manual'`, the default, nothing is dropped.
Once due, `up` and `check` warn about it:

```
[WARN] [release] the deprecated table "legacy_orders" is due for dropping, deprecated 4 releases ago. Drop it in a migration with db.dropDeprecated("legacy_orders").
```

Drop it in a v2 migration:

```js
// everything due
await db.dropDeprecated();
// one table or column, due or not
await db.dropDeprecated('legacy_orders');
await db.dropDeprecated('users', 'fax');
```

With `drop: 'auto'` db-migrate drops it on its own after `N` releases. Set
the options per call or for the project:

```js
await db.deprecateTable('legacy_orders', { releases: 2, drop: 'auto' });
```

```json
{
  "deprecation": {
    "releases": 2,
    "drop": "auto"
  }
}
```

The project options go into `.db-migraterc`, or `cmdOptions` of the
[programmable API](../API/programable.md).

## Reverting

Reverting the migration deprecating a table or column removes the mark,
and relaxing `notNull` is reverted. Reverting the first migration of a
release reverts what was done before it, a renamed table or column gets its
name back, a dropped one is created again, without its data.

`db-migrate fix` learns the renaming and dropping of the releases again,
since db-migrate 1.6.0.

## Rows deleted in soft mode

A [soft delete](data migrations.md#soft-delete) keeps the rows, until they
are purged. With `purge`, they are purged along the releases as well:

```js
await db.delete('sessions', { expired: true }, {
  mode: 'soft',
  column: 'deleted_at',
  purge: { releases: 2, drop: 'auto' }
});
```

`purge: true` takes the options of the project. Once due, the rows are
purged before the first migration of the release with `drop: 'auto'`, or
warned about with `drop: 'manual'` until `db.purge` purges them. A release
which purged rows can not be reverted, `down` refuses it before reverting
anything.
