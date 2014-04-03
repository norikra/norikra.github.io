---
layout: layout
title: Query of Norikra
subtitle: Syntax and examples
---
# Query of Norikra

Norikra's query is actually [Esper's EPL](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/index.html). But some limitations and additional features exist.

* Limitations
  * Only `SELECT` clause is supported and tested
    * Features for SELECT are supported: Sub queries, JOINs and Patterns
    * Java method invocations
    * UDFs
  * Unsupported statements of EPL
    * Annotations
    * Additional declarations (expressions, scripts, schemas and windows)
    * Contexts, variables and constants
    * Merging/splitting/updating streams (`INSERT INTO`, `UPDATE`)
    * Accessing external data
    * Event delivery with `For` clause
* Additional features
  * Schema-less events (called 'targets')
  * Complex events support: Container fields (hash, array)

Norika's queries are designed to do simple `SELECT ... FROM ...`, without any schemas. Many of complex features of EPL can be done with schema-less simple SELECT queries (ex: Merging streams are same with to use a target for many kind of events).

Norikra's queries MUST be parsed and compiled as Esper EPL. Extended syntax is in EPL syntax.

## Query names

All queries have a name and a group name.

* Name
  * must be unique in all queries
  * is key to fetch results of query
* Group
  * may be blank or a string
  * is key to sweep results of query set

Names and groups are not accessed in query, and their naming rule is not defined (any strings are valid).

## Syntax

Norikra's query is a kind of standard SQL, except for views. SQL keywords and functions are case insensitive, but **target and field names are case sensitive**.

```sql
SELECT
  age,
  COUNT(1, session.valid IS NOT NULL) AS login_users,
  COUNT(*) AS users
FROM accesses.win:time_batch(1 hours)
WHERE name.length() > 0
GROUP BY age
```

This query will be evaluated for only event records with `name`, `age` and `session.valid` fields.

## Target names

Target is something like a table name of RDBMS.
But target is a name of set of streams, which have different set of fields. In typical way, one target is for one web service or one customer.

Target naming rule is `[a-zA-Z][_a-zA-Z0-9]*`.

## Views (Windows)

Views are used to define time windows or to control timing to fire events. With views, norikra controls the range of events for each queries.

### No views specified

A query without any views specified runs for all events. This query puts output events as much as events with age over 15.

```sql
SELECT name, age FROM events WHERE age > 15
```

Norikra v0.1.x has no RPC API to fetch output events as stream. So queries of this type are slightly dangerous: huge amount of output events may exist in memory if user does not fetch these.

### Time window (time batch window)

Time batch windows (`win:time_batch( time-period )`) separate events by its arrival time, and does 'group by' by this separation. Output events are generated just once per intervals of specified time period.

This specifier may be a most major one for Norikra.

```sql
SELECT
  name,
  age,
  COUNT(*) AS count
FROM events.win:time_batch(1 hours)
WHERE age > 15
GROUP BY name, age
```

This query does its process for 1 hour from the time query registered, to 1 hour after it. Output events appears on just 1 hour after.

```
10:31:05  query registered
10:31:06  event 1
10:35:51  event 2
.
.
11:31:04  event N
11:31:05  -> output events (for event 1 .. N)
11:31:05  event N+1
.
.
12:31:05  -> output events (for event N+1 .. )
```

For time-period, many units are available, like `sec|second|seconds, min|minute|minutes, hour|hours, day|days` to `year|years` !. ([EPL document](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/epl_clauses.html#epl-syntax-time-periods))

If you want to specify time period offset to any point (ex: from 10:00:00 to 11:00:00 for 1 hour time batch), use second (optional) argument of time_batch with long-value milliseconds from epoch (ex: `events.win:time_batch(1 hour, 0L)`).

And third optional argument is flow control. Comment below is a quotation from Esper EPL reference.

    By default, if there are no events arriving in the current interval (insert stream), and no events remain from the prior batch (remove stream), then the view does not post results to listeners. The view allows overriding this default behavior via flow control keywords.

If no one event arrived in a time batch, query outputs '0' records only for the first time, and no records for second (or later) time batches. For available options, see [EPL document](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/epl-views.html#view-win-time-batch).

### Externally timed batch window

Externally timed batch window (`win:ext_timed_batch( expression, time-period )`) is same with time batch window, except for its 'time'. This window uses any fields (or a valid expression) instead of system time of event arrival (time batch window).

```sql
SELECT
  name,
  age,
  COUNT(*) AS count
FROM events.win:time_batch(timestamp, 1 hours)
WHERE age > 15
GROUP BY name, age
```

`timestamp` must be a long-value field (Norikra's 'integer' fields are actually Long) which contains a milliseconds from epoch. Expressions are valid if you have a field with seconds from epoch. (ex: `FROM events.win:time_batch( seconds_from_epoch * 1000, 1 hours )`)

Optional arguments for time offset and flow controls are also available.

### Time length window (sliding time window)

Time window (`win:time( time-period )`), from specified time ago of latest event arrival, to latest event. Output events are generated for every input events.

```sql
SELECT
  name,
  age,
  COUNT(*) AS count
FROM events.win:time(1 hours)
WHERE age > 15
GROUP BY name, age
```

This query generates output events as much as input events (as same as no views specified).

### Length (or Length batch) window

Length window (`win:length( size )`) or length batch window (`win:length_batch( size )`) are works you expecting for event sizes.

### Unique

The unique view (`std:unique( expression [, expression ... ] )`) for expressions. Queries with this view works when events arrived and specified expressions returns unique value(s).

```sql
SELECT
  family_name,
  name,
  age
FROM events.std:unique(name, family_name)
WHERE age > 15
```

This query generates output events for the first arrival time of each name-family_name combinations. Of course, this distinct calculations are executed on memory, so we should use this view in combination with time (or other) windows (except for special cases, ex:country names).

* Conbination

Views can be combined. The query below is used to get the first events of each event_code, for each days.

```sql
SELECT
  event_code,
  parameter
FROM events.win:time_batch(1 day, 0L).std:unique(event_code)
```

* Others

[Many more views](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/epl-views.html) are availabe in Norikra, as same as Esper EPL itself. Norikra have no limitations for views. If you found any unavailabilities, it's bug.

## Field name escape

Field names are written as `[_a-zA-Z0-9]+` or dot-separated these parts in queries. (Dot-separated field names are container field accesses. See following 'Container fields' section.)

Norikra's input events are object (or hash, map, ...) that have pairs of field names and values. Its field names are directly referred by field names in queries.

Input event:

```js
{
  "username": "tagomoris",
  "email": "tagomoris@gmail.com",
  "url": "https://github.com/tagomoris",
  "repositories": 50
}
```

Query example:

```sql
SELECT
  username,   -- "tagomoris"
  email,       -- "tagomoris@gmail.com"
  repositories -- 50
FROM events
```

All norikra fields have a type, one of 'string', 'boolean', 'integer'(defined as 'long' internally), 'float'(defined as 'double' internally) or 'hash' and 'array'. 'hash' and 'array' are container fields.
Types of fields are specified when targets are opened, or determined automatically by value types of the first input event for lazy targets.

To Access fields of input event record with invalid characters as query's field name, escape invalid chars with underscores (`_`).

Input event:

```js
{
  "user name": "tagomoris",
  "email": "tagomoris@gmail.com",
  "url": "https://github.com/tagomoris",
  "repositories": 50,
  "phone.number": "+81-090-xxxx-xxxx"
}
```

Query example:

```sql
SELECT
  user_name,   -- "tagomoris"
  phone_number -- "+81-090-xxxx-xxxx"
FROM events
```

Different fields with same escaped names (ex: "user.name", "user-name" and "user name") are mixed on Norikra's query, and treated as one field. If it is not desired behavior for you, you should escape fields before sending norikra server.

## Container fields

For 'hash' and 'array' fields, event records have nested objects, and we can access these nested objects by dot-separated field names.

Input event:

```js
{
  "user": {
    "name": "tagomoris",
    "email": "tagomoris@gmail.com",
  },
  "visit": "Tokyo",
  "route": ["Shibuya", "Ohsaki", "Shinjuku", "Ikebukuro"]
}
```

Query example:

```sql
SELECT
  user.name, -- "tagomoris"
  route.$0   -- "Shibuya"
FROM events
WHERE visit="Tokyo" AND user.email IS NOT NULL
```

Accessing hash member is just joined fields by dot, like `user.name`. To specify array member, use index as `$0`, `$1`, ... (negative index is not supported yet). Of course, 3 or more nested field accesses are available: `users.$1.data.name.first`.

Use `$$0` to specify string representations of integer (like '0') for hash member field name.

## Java methods, operators and functions

In norikra (and Esper) queries, each fields' values are Java object (String, Boolean, Long, Double), and we can call these instance methods on queries.

```sql
SELECT
  log.substring(8) AS message
FROM events
WHERE log.startsWith('WARNING:')
```

And some of Java packages are automatically imported, so we can also call functions like `Math.abs()`.

    Esper auto-imports the following Java library packages:
    java.lang.*
    java.math.*
    java.text.*
    java.util.*

Method chains are also available, ex:

```sql
SELECT
  DateFormat.getInstance().parse(date_field).getTime() AS epoch_msec
FROM events
```

Norikra/Esper has many operators:

* Arithmetic operations: `+`,`-`, `*`, `/`, `%`
* Logical/comparison operations: `NOT`, `OR`, `AND`, `=`, `!=`, `<`, `>`, `<=`, `>=`, `is`, `is not`
* String concatenation operation: `||`
* String comparison operations: `LIKE`, `REGEXP`
* `exp IN (items)`, `exp IN rage`(open range: `(0:100)` and close range: `[0:100]`), and `between`

See [EPL reference: Operators](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/epl-operator.html).

Esper has built-in functions like these:

* `CASE value WHEN v1 THEN r1 WHEN v2 THEN r2 [...] ELSE rr END`
* `CASE WHEN cond1 THEN r1 [...] ELSE rr END`
* `CAST( expression, type_name )`
* `PREV()`, `CURRENT_TIMESTAMP()`
* `AVG()`, `COUNT()`, `MAX()`, `MIN()`, `SUM()` and DISTINCT
* `FIRST()`, `LAST()`, `MAXBY()`, `MINBY()`

See [EPL reference: Functions](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/functionreference.html).

## Group by, Having, Order by, Limit

These keywords work as same as we expect. For more, see EPL reference.

* "Aggregates and grouping: the Group-by Clause and the Having Clause" ([link](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/epl_clauses.html#epl-grouping-aggregating))
* "Limiting Row Count: the Limit Clause" ([link](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/epl_clauses.html#epl-limit))
* "Sorting Output: the Order By Clause" ([link](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/epl_clauses.html#epl-order-by))

## JOINs and SubQueries

JOINs and SubQueries are available, but Norikra requires fully-qualified field names for JOINs/SubQueries.

Example query (JOIN, default is inner-join):

```sql
SELECT
  a.name AS name,
  a.country AS country,
  b.code AS event_code,
  COUNT(*) AS times
FROM registrations.std:unique(name) AS a,
     behavior.win:time_batch(1 hours) AS b
ON a.name = b.name
WHERE b.log_level IN ('INFO', 'WARN', 'ERROR')
GROUP BY a.name, a.country, b.code
```

Example query (SubQuery):

```sql
SELECT
  name,
  ( SELECT a.country
    FROM registrations.std:unique(name) AS a
    WHERE a.name = behavior.name
  ) AS country,
  code
FROM behavior
WHERE log_level IN ('INFO', 'WARN', 'ERROR')
```

"Fully-qualified field name" means specifier like `TARGET.FIELD` or `ALIAS.FIELD`. For JOINs, Norikra cannot deterine the target fields belongs to, so all fields must be specified with target name or alias. For SubQueries, fields in inner-query must be specified with targets/aliases. Fields of outer-query does not require it.

For JOINs and SubQueries, Esper EPL have many restrictions. See documents.

* "Joining Event Streams" ([link](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/epl_clauses.html#epl-join))
* "Subqueries" ([link](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/epl_clauses.html#epl-subqueries))

## Fully-qualified field names and container fields

In norikra query, all fields can be qualified with targets (ex: `SELECT Events.code FROM Events ...`). But Norikra cannot determine whether first part is a target name or a container field name if hash/array container field have a same name with target.

```sql
-- TARGET Name: Events
-- Data:
--  {
--    "Events": {"code":404, "path":"/"},
--    "code": 0,
--  }
SELECT
  Events.code -- 404 ? 0 ?
FROM Events
```

In these case, Norikra handle 'Events' as target name. To access the field with the same name of target, use fully-qualified field name:

```sql
SELECT
  Events.Events.code, -- 404
  Events.Events.path, -- "/"
  code                -- 0
FROM Events
```

## Filters/Patterns

Both of [Filters](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/epl_clauses.html#epl-from-clause-filter) and [Patterns](http://esper.codehaus.org/esper-4.11.0/doc/reference/en-US/html/event_patterns.html) are available in Norikra. For more details, see EPL's reference.

## Miscellaneous rules

### Selecting all fields

In norikra query, `SELECT * FROM ...` may NOT perform as user expects. Specify all fields required in explicit way. This is a restriction of Norikra's schema-less feature.

### Backquote escapes

Esper EPL's syntax have escaping by backquotes. But Norikra's query does NOT support it. In norikra queries, use field names escaped by underscore instead of backquote escaping.

    NG: `field name`
    OK: field_name

### Field existing check

Norikra's queries are evaluated for events, which have all fields used in that query. Field existence checks are automatically done by Norikra server, and events without required fields are ignored.

Esper EPL have a function named as `exist()`, but in Norikra query, `exist( someone )` referes a field 'someone', so this expression is always true. So you should just use fields directly, without existence check, in Norikra.

If you want to use any field names in query even though that fields does not exists in input records, specify that field as non-optional.
