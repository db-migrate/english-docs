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
[migration lock](running in parallel.md).

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

Jobs run without the migration lock, alongside other migrations. Do not
change the schema of tables a running job works on, or make the job
blocking and wait for it.
