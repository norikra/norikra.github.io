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
