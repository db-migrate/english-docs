# Writing plugins

A plugin is an npm package whose name starts with `db-migrate-plugin`. Its main
module exports the hooks it implements and a `loadPlugin` function, which
db-migrate calls once before the first hook is used:

```js
module.exports = {
  name: 'example',
  hooks: ['init:config:overwrite:require'],

  loadPlugin: function () {
    module.exports = Object.assign(module.exports, {
      'init:config:overwrite:require': function (fileName) {
        // delay requires to here, so the plugin does not slow down
        // db-migrate when it is not used
        const yaml = require('js-yaml');
        return yaml.load(require('fs').readFileSync(fileName, 'utf8'));
      }
    });

    delete module.exports.loadPlugin;
  }
};
```

db-migrate loads the plugins listed in `dependencies` and `devDependencies` of
the `package.json` in the current directory, from its `node_modules`. A plugin
without `hooks` or `loadPlugin` is skipped. Its `name` is replaced by the
package name. `loadPlugin` is called whenever one of its hooks is about to be
used, as long as it exists, so it deletes itself as shown.

Some hooks are used by every plugin implementing them, others only by the first
one, which is noted below. If several plugins implement such a hook, db-migrate
warns and uses the first.

## Hooks

### init:config:overwrite:require (first plugin)

`(fileName) => config`

Reads the database config file instead of db-migrate, e.g. to support another
format.

### file:hook:require (every plugin)

`() => { extensions, load }`

Adds migration files with other extensions. `extensions` is a regular
expression alternative of extensions without the dot, like `'sql'` or
`'ts|mts'`.

Without `load`, the files are loaded with `require`, so the plugin has to make
`require` handle them (e.g. by registering a compiler). With `load`, db-migrate
calls `load(path)` and expects the migration module back, like `require` would
return it: `{ up, down }` for a v1 migration or `{ migrate, _meta }` for a v2
migration.

The older name of this hook, `migrator:migration:hook:require`, still works,
but db-migrate warns that the plugin is outdated.

### create:template (every plugin)

Offers a template for `db-migrate create`, chosen by an option on the command
line or in the config. The plugin exports:

- `'init:template'`: `() => ({ option, type })`. The template is used when
  `--<option>` is passed or `option` is set in the config. The first plugin
  whose option is set wins. Without custom write, db-migrate writes
  `<timestamp>-<name>.js` with the content of
  `template:overwrite:provider:<type>`.
- `'create:template:custom:write'`: `true` to write the file itself.
- `'write:template'`: `(opts, write) => Promise`, with custom write only.
  `opts` holds copies of the options (`argv`) and the config (`config`).
  Calls `write({ extension, type, suffix, pathExtension })` to create the
  migration file from the template `type`: `extension` (default `.js`) and
  `suffix` are appended to the file name, `pathExtension` to the directory.
  It can be called several times, e.g. for several files.

### template:overwrite:provider:&lt;type&gt; (first plugin)

`({ file: { name } }) => string`

The content of new migration files of the template `type`.

### connection:tunnel:&lt;type&gt; (first plugin)

`(tunnelConfig) => Promise`

Opens a tunnel before connecting, for an environment with a `tunnel` section.
`<type>` is `tunnel.tunnelType`, `ssh` by default. The tunnel config has
`dstHost` and `dstPort` set to the database, db-migrate connects to
`127.0.0.1:localPort` once the promise resolved. The tunnel is opened once and
shared by all connections of db-migrate.

### run:default:action:&lt;action&gt;:overwrite (first plugin)

`(internals, config) => void`

Implements a new command, `db-migrate <action>`. `internals.argv` holds the
parsed options, its `_` the arguments after the command, and `config` the
loaded configuration, `config.getCurrent()` returning `{ env, settings }` of
the current environment.

### init:api:addfunction:hook (every plugin)

`() => [name, fn]`

Adds the function `fn` as `name` to the programmable API. Plugins are
registered on it by `registerAPIHook()`.

## Examples

- [db-migrate-plugin-sql](https://github.com/db-migrate/plugin-sql):
  `file:hook:require` with `load`, `create:template` with custom write
- [db-migrate-plugin-tunnel-ssh](https://github.com/db-migrate/plugin-tunnel-ssh):
  `connection:tunnel:ssh`
- [db-migrate-plugin-yaml](https://github.com/db-migrate/plugin-yaml):
  `init:config:overwrite:require`
