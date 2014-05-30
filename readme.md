# SQL Bricks.js

[![Build Status](https://travis-ci.org/CSNW/sql-bricks.png?branch=master)](https://travis-ci.org/CSNW/sql-bricks)

SQL Bricks.js is a transparent, schemaless library for building and composing SQL statements.

- Supports all SQL-92 clauses for select/insert/update/delete (plus some postgres & sqlite additions)
- Over [150 tests](http://csnw.github.io/sql-bricks/browser-tests.html)
- Easy-to-use, comprehensive [docs](http://csnw.github.io/sql-bricks)
- Single straightforward [source file](sql-bricks.js) (less than 1,000 lines), easy to understand & debug


Comparison with popular SQL-generation libraries:

library         | lines | files | schema       | language     | other notes  
--------------- | ----- | ----- | ------------ | --------     | --------------
[Knex][1]       | 3500  |    30 | schema       | javascript   | transactions, migrations, promises, connection pooling
[Squel][2]      | 1000  |     3 | schemaless   | coffeescript | 
[node-sql][3]   | 2600  |    59 | schema       | javascript   |
[mongo-sql][4]  | 1700  |    49 | schemaless   | javascript   | 
[gesundheit][5] | 1600  |    21 | schemaless   | coffeescript | uses Any-DB to wrap the DB driver
[sql-bricks][6] |  800  |     1 | schemaless   | javascript   |

[1]: https://github.com/tgriesser/knex
[2]: https://github.com/hiddentao/squel
[3]: https://github.com/brianc/node-sql
[4]: https://github.com/goodybag/mongo-sql
[5]: https://github.com/BetSmartMedia/gesundheit
[6]: https://github.com/tgriesser/knex

# Use

SQLBricks' only dependency is [Underscore.js](http://underscorejs.org/).

In the browser:

```javascript
var select = SqlBricks.select;

select().from('user').where({last_name: 'Rubble'});
// SELECT * FROM user WHERE last_name = 'Rubble'
```

In node:

```javascript
var select = require('sql-bricks').select;

select().from('user').where({last_name: 'Rubble'});
// SELECT * FROM user WHERE last_name = 'Rubble'
```

# Examples

The [SQLBricks API](http://csnw.github.io/sql-bricks/) is comprehensive, supporting all of SQL-92 for select/insert/update/delete. It is also quite flexible; in most places arguments can be passed in a variety of ways (arrays, objects, separate arguments, etc). That said, here are some of the most common operations:

```javascript
// WHERE: (.toString() is optional; JS will call it automatically in most cases)
select().from('user').where({last_name: 'Rubble'}).toString();
// SELECT * FROM user WHERE last_name = 'Rubble'

// JOINs:
select().from('user').join('address').on({'user.addr_id': 'address.id'});
// SELECT * FROM user INNER JOIN address ON user.addr_id = address.id

// Nested WHERE criteria:
select('*').from('user').where(or(like('last_name', 'Flint%'), {'first_name': 'Fred'}));
// SELECT * FROM user WHERE last_name LIKE 'Flint%' OR first_name = 'Fred'

// GROUP BY / HAVING
select('city', 'max(temp_lo)').from('weather')
  .groupBy('city').having(lt('max(temp_lo)', 40))
// SELECT city, max(temp_lo) FROM weather
// GROUP BY city HAVING max(temp_lo) < 40

// INSERT
insert('user', {'first_name': 'Fred', 'last_name': 'Flintstone'});
// INSERT INTO user (first_name, last_name) VALUES ('Fred', 'Flintstone')

// UPDATE
update('user', {'first_name': 'Fred', 'last_name': 'Flintstone'});
// UPDATE user SET first_name = 'Fred', last_name = 'Flintstone'


// Parameterized SQL
update('user', {'first_name': 'Fred'}).where({'last_name': 'Flintstone'}).toParams();
// {"text": "UPDATE user SET first_name = $1 WHERE last_name = $2", "values": ["Fred", "Flintstone"]}

// SQLite-style params
update('user', {'first_name': 'Fred'}).where({'last_name': 'Flintstone'}).toParams({placeholder: '?%d'});
// {"text": "UPDATE user SET first_name = ?1 WHERE last_name = ?2", "values": ["Fred", "Flintstone"]}

// MySQL-style params
update('user', {'first_name': 'Fred'}).where({'last_name': 'Flintstone'}).toParams({placeholder: '?'});
// {"text": "UPDATE user SET first_name = ? WHERE last_name = ?", "values": ["Fred", "Flintstone"]}
```

Documentation: http://csnw.github.io/sql-bricks

License: [MIT](LICENSE.md)
