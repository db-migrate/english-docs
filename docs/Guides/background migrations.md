# Background migrations

A [data migration](data migrations.md) changing millions of rows can take
hours. As a background migration, it does not hold up the deployment: `up`
only registers it as a job and goes on with the following migrations, your
application starts, and runs the jobs in the background with `executeWork`.

```js
exports.migrate = async db => {
  await db.update('orders', { currency: 'EUR' }, { currency: null });
  await db.delete('sessions', ['created < ?', ['2024-01-01']], {
    mode: 'soft',
    column: 'deleted_at'
  });
};

exports._meta = {
  version: 2,
  type: 'dml',
  background: true
};
```

Background migrations need db-migrate 1.4.0 and a driver with the
[migration lock](running in parallel.md). Since 1.4.1 the jobs pause while
migrations run, see [Jobs and migrations](#jobs-and-migrations).

## Running the jobs

```js
const DBMigrate = require('db-migrate');
const dbm = DBMigrate.getInstance(true);

const worker = dbm.executeWork({
  parallel: 2,
  pause: 50,
  batch: 500,
  watch: true
});

// on shutdown
await worker.stop();
```

| Option | Default | |
|---|---|---|
| `parallel` | 1 | jobs run at once by this worker, each on connections of its own |
| `pause` | 0 | ms to wait between two batches of a job, to keep the load low |
| `batch` | 1000 | rows per batch, unless the instruction sets its own `batch` |
| `watch` | false | keep looking for new jobs, instead of ending once none is left |
| `interval` | 5000 | ms between looking for new jobs |
| `timeout` | 60000 | ms after which the job of a worker which stopped responding is taken over |

`executeWork` returns right away with `{ done, stop }`. `done` resolves with
`{ done, failed }`, the names of the jobs done and the jobs failed with
their error, once no job is left, or after `stop()`. `stop()` lets the
running jobs stop after their current batch, they continue with the next run.

## Several instances

Any number of instances can call `executeWork` at once, e.g. every instance
of your application. Each takes one job at a time per `parallel`, a job runs
on one instance only. The jobs are taken in the order of their migrations.

A worker renews its jobs while running them. If an instance dies, its jobs
are taken over by another instance after `timeout` and continued where they
stopped, like an [interrupted run](failures and recovery.md#interrupted-runs-v2).
The worker taking over has to be running, start it with `watch: true` to keep
it looking.

## Blocking jobs

Jobs run independently of each other. If a job has to be done before the
jobs after it start, mark it as blocking:

```js
exports._meta = {
  version: 2,
  type: 'dml',
  background: true,
  blocking: true
};
```

The jobs of later migrations wait until it is done.

## The state of a job

A background migration counts as run once its job is done. Until then it is
running: `up` and `check` log it as running in the background and do not run
it again. Once done, it is reverted by `down` like any data migration.

If a job fails, it is rolled back like a failing data migration and marked as
failed, workers do not take it again. The next `db-migrate up` queues it again,
fix the migration first. A failing job with steps which can not be reverted is
not rolled back and continues after them.

## Reverting

`down` takes the jobs as the latest migrations, they come first, before the
migrations recorded as run. A job running or stopped half way is paused,
the steps it executed so far are reverted, and the job is forgotten. A job
not started yet is only forgotten. Once done, a background migration is
reverted like any data migration.

Reverting a job restores the rows it changed so far, from its backup tables
or by its marks, in the foreground and while holding the migration lock. For
a large job this can take as long as running it did, and other migrations
and the jobs wait meanwhile. Since db-migrate 1.4.1, before `down` missed the
jobs.

## Jobs and migrations

Migrations take precedence over the jobs. Whenever `up`, `down` or `fix` has
something to run, it pauses the jobs after taking the
[migration lock](running in parallel.md): no job is taken anymore, the
running ones stop after their current batch, and the migrations wait for
them:

```
[INFO] [jobs] waiting for 2 background job(s) to pause: 20261009120000-orders, 20261009130000-sessions
```

Once the migrations are done and the lock is released, the workers continue
the jobs where they stopped. So a migration never changes a table while a job
works on it, and a deployment waits at most for the batches running at that
moment.

The pause lasts as long as the migrating process holds the lock. If it dies,
the jobs continue once its lock is considered stale, after `--lock-timeout`.
A job whose worker does not respond for the lock timeout is not waited for,
it is continued by another worker later.
