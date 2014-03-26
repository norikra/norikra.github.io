---
layout: layout
title: Tips - Norikra
subtitle: Tips to write norikra queries
---
# Tips

## Esper query syntax

### Counts and example by a query

* `LAST()` or `FIRST()` available

```sql
SELECT level, type, LAST(message) AS message, COUNT(*) AS count
FROM eventA.win:time_batch(1 min)
WHERE level IN ('ERROR', 'WARN')
GROUP BY level, type
```

### Operators

* String concatination operator: `||`
  * ex: `SELECT (str1 || str2) AS joind_str FROM TargetA WHERE ...`

### Aggregatin functions with filters

Esper's aggregation functions (ex: `count()`, `sum()`, ...) have optional 2nd argument, filters. For example, this query calculates sumation of `bytes` of records only with `status: 200`, and bytes of all records.

```sql
SELECT
 SUM(bytes, status=200) assuccess_bytes,
 SUM(bytes) as total_bytes
FROM access_log.win:time_batch(1 hour)
```

NOTE: For `count()` with filters, don't use `*` for 1st argument. Esper compiles `SELECT COUNT(*, x=y) FROM ...` as `SELECT COUNT(x=y) FROM ...`. This may not what we want. Use `SELECT COUNT(1, x=y) FROM ...`. (This may be a bug of esper.)

## Query/Target persistence

Use `--stats` and related options of `norikra start`.

* `--stats PATH`
  * JSON document, stores:
    * configurations: listen address/port, threads, logging
    * targets and queries
  * loaded on process bootstrapping (missing: ignored)
  * stored on process shutdown (overwrite current file)
* `--suppress-dump-stat`
  * specify not to store stats on process shutdown
* `--dump-stat-interval=N`
  * store stats dump per specified N seconds (ex: 1800)

Stats will be dumped with SIGUSR1 if `--stats PATH` specified.
