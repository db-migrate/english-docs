[![Build Status](https://github.com/db-migrate/node-db-migrate/actions/workflows/ci.yml/badge.svg?branch=master)](https://github.com/db-migrate/node-db-migrate/actions/workflows/ci.yml)

# db-migrate

Database migration framework for node.js

Officially supported is Node.js 24 and newer. Coming from db-migrate 0.11? See
[Upgrading from 0.11](Getting Started/upgrading.md).

## _Official_ Supported Databases

- PostgreSQL - npm i db-migrate-pg, see [PostgreSQL](Drivers/pg.md)
- MySQL and MariaDB - npm i db-migrate-mysql, see [MySQL](Drivers/mysql.md)
- sqlite3 - npm i db-migrate-sqlite3, see [sqlite3](Drivers/sqlite3.md)
- CockroachDB - npm i db-migrate-cockroachdb, see [CockroachDB](Drivers/cockroachdb.md)
- MongoDB - npm i db-migrate-mongodb, see [MongoDB](Drivers/mongodb.md)

## Getting started

1. [Install](Getting Started/installation.md) db-migrate and your driver.
2. [Configure](Getting Started/configuration.md) your database.
3. [Create and run](Getting Started/usage.md) migrations, see all
   [commands](Getting Started/commands.md).

## License

(The MIT License)

Copyright (c) 2015 Jeff Kunkle, Tobias Gurtzick

Copyright (c) 2013 Jeff Kunkle

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
