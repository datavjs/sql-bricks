# SQL Bricks.js 

[![Build Status](https://travis-ci.org/CSNW/sql-bricks.svg?branch=master)](https://travis-ci.org/CSNW/sql-bricks)

SQL Bricks.js is a transparent, schemaless library for building and composing SQL statements.

- The core library supports all [SQL-92](https://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt) clauses for `SELECT`, `INSERT`, `UPDATE` and `DELETE` (with the exception of asc/desc/collate options for `orderBy()`, see [#73](https://github.com/CSNW/sql-bricks/issues/73))
  - Postgres extensions are at https://github.com/Suor/sql-bricks-postgres
  - MySQL extensions are at https://github.com/tamarzil/mysql-bricks
  - SQLite extensions are at https://github.com/CSNW/sql-bricks-sqlite
- Over [200 tests](https://datavjs.github.io/sql-bricks/browser-tests.html)
- Easy-to-use, comprehensive [docs](https://datavjs.github.io/sql-bricks)
- Single [source file](sql-bricks.js) (~1,100 lines)
- No production dependencies and only 1 dev dependency (Mocha.js)

Comparison with other SQL-generation JS libraries:

library         | lines | files | schema     | other notes  
--------------- | ----- | ----- | ---------- | --------------
[Knex][1]       | 20k   |   ~50 | schema     | transactions, migrations, promises, connection pooling
[Squel][2]      | 1.7k  |     1 | schemaless |
[node-sql][3]   | 2.6k  |   ~60 | schema     |
[mongo-sql][4]  | 1.7k  |   ~50 | schemaless |
[sql-bricks][5] | 1.1k  |     1 | schemaless |

[1]: https://github.com/tgriesser/knex
[2]: https://github.com/hiddentao/squel
[3]: https://github.com/brianc/node-sql
[4]: https://github.com/goodybag/mongo-sql
[5]: https://github.com/CSNW/sql-bricks

# Related Libraries

* [mysql-bricks](https://github.com/tamarzil/mysql-bricks) adds mysql-dialect extensions:
  * `INSERT ... ON DUPLICATE KEY UPDATE ...`
  * `INSERT IGNORE ...`
  * `LIMIT (SELECT / UPDATE / DELETE)`
  * `OFFSET`
  * `ORDER BY (UPDATE / DELETE)`
* [sql-bricks-sqlite](https://github.com/CSNW/sql-bricks-sqlite) adds sqlite-dialect extensions:
  * `LIMIT` and `OFFSET`
  * `OR REPLACE`, `OR ABORT`, `OR ROLLBACK`, `OR FAIL`
* [sql-bricks-postgres](https://github.com/Suor/sql-bricks-postgres) adds postgres-dialect extensions:
  * `LIMIT` and `OFFSET`
  * `RETURNING`
  * `UPDATE ... FROM`
  * `DELETE ... USING`
  * `FROM VALUES`
* [pg-bricks](https://github.com/Suor/pg-bricks) adds:
  * connections
  * transactions
  * query execution
  * data accessors
* [A Layer Above Database Connectors](https://github.com/paleo/ladc) adds:
  * A common way to access to relational databases (SQLite & Postgres as of Oct 2019)
  * A pool of connections in order to allow transactions in an asynchronous context;
  * A way to augment your connector with your SQL query builder (has a sql-bricks plugin)

# Use

In the browser:

```javascript
var select = SqlBricks.select;
```

In node:

```javascript
var select = require('sql-bricks').select;
```

A simple select via `.toString()` and `.toParams()`:

```javascript
select().from('person').where({last_name: 'Rubble'}).toString();
// "SELECT * FROM person WHERE last_name = 'Rubble'"

select().from('person').where({last_name: 'Rubble'}).toParams();
// {"text": "SELECT * FROM person WHERE last_name = $1", "values": ["Rubble"]}
```

While `toString()` is slightly easier, `toParams()` is recommended because:

* It provides [robust protection](https://en.wikipedia.org/wiki/SQL_injection#Parameterized_statements) against SQL injection attacks (toString() just does basic escaping)
* It provides better support for complex data types (objects, arrays, etc, are passed directly to your database driver instead of being "stringified")
* It provides more helpful error messages (see https://github.com/Suor/sql-bricks-postgres/issues/10)

# Examples

The [SQLBricks API](https://datavjs.github.io/sql-bricks/) is comprehensive, supporting all of SQL-92 for select/insert/update/delete. It is also quite flexible; in most places arguments can be passed in a variety of ways (arrays, objects, separate arguments, etc). That said, here are some of the most common operations:

```javascript
// convenience variables (for node; for the browser: "var sql = SqlBricks;")
var sql = require('sql-bricks');
var select = sql.select, insert = sql.insert, update = sql.update;
var or = sql.or, like = sql.like, lt = sql.lt;

// WHERE: (.toString() is optional; JS will call it automatically in most cases)
select().from('person').where({last_name: 'Rubble'}).toString();
// SELECT * FROM person WHERE last_name = 'Rubble'

// JOINs:
select().from('person').join('address').on({'person.addr_id': 'address.id'});
// SELECT * FROM person INNER JOIN address ON person.addr_id = address.id

// Nested WHERE criteria:
select('*').from('person').where(or(like('last_name', 'Flint%'), {'first_name': 'Fred'}));
// SELECT * FROM person WHERE last_name LIKE 'Flint%' OR first_name = 'Fred'

// GROUP BY / HAVING
select('city', 'max(temp_lo)').from('weather')
  .groupBy('city').having(lt('max(temp_lo)', 40))
// SELECT city, max(temp_lo) FROM weather
// GROUP BY city HAVING max(temp_lo) < 40

// INSERT
insert('person', {'first_name': 'Fred', 'last_name': 'Flintstone'});
// INSERT INTO person (first_name, last_name) VALUES ('Fred', 'Flintstone')

// UPDATE
update('person', {'first_name': 'Fred', 'last_name': 'Flintstone'});
// UPDATE person SET first_name = 'Fred', last_name = 'Flintstone'


// Parameterized SQL
update('person', {'first_name': 'Fred'}).where({'last_name': 'Flintstone'}).toParams();
// {"text": "UPDATE person SET first_name = $1 WHERE last_name = $2", "values": ["Fred", "Flintstone"]}

// SQLite-style params
update('person', {'first_name': 'Fred'}).where({'last_name': 'Flintstone'}).toParams({placeholder: '?%d'});
// {"text": "UPDATE person SET first_name = ?1 WHERE last_name = ?2", "values": ["Fred", "Flintstone"]}

// MySQL-style params
update('person', {'first_name': 'Fred'}).where({'last_name': 'Flintstone'}).toParams({placeholder: '?'});
// {"text": "UPDATE person SET first_name = ? WHERE last_name = ?", "values": ["Fred", "Flintstone"]}
```

Full documentation: https://datavjs.github.io/sql-bricks

License: [MIT](LICENSE.md)
