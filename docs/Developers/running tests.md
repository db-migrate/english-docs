# Running Tests

The tests of db-migrate use [lab](https://hapi.dev/module/lab/), `npm test`
lints the code with eslint first:

```bash
npm install
cp test/db.config.example.json test/db.config.json
npm test
```

Adjust `test/db.config.json` to the database instances you test against. The
drivers have their own test suites in their repositories.
