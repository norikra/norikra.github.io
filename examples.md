---
layout: layout
title: Query Examples
subtitle: Design patterns for Norikra queries
---
# Query Examples

TODO: more examples and patterns

## Counting

### Count impressions and unique users

Just do with `DISTINCT` as same as RDBMS.

```sql
SELECT COUNT(DISTINCT cookie.id) AS uu,
       COUNT(*) AS imps
FROM impressions.win:time_batch(1 hours)
WHERE cookie.valid
```

### Count logs of each patterns

We want to count access logs of specified path per 10 minutes. Total, successes and internal server errors are required.

```sql
SELECT
  path,
  COUNT(1, status=200) AS success_count,
  COUNT(1, status=500) AS servererror_count,
  COUNT(*) AS count
FROM AccessLog.win:time_batch(10 min, 0L)
WHERE service='myservice' AND path LIKE '/api/%'
GROUP BY path
```

Esper's `COUNT()` can receive optional 2nd argument as a filter expression, as filter for countings. This 2nd optional argument are also exists for `AVG()`, `SUM()` and `FMAX()`, `FMIN()`.

NOTE: When using `COUNT()` with second argument, DO NOT USE `*` for first argument. `COUNT(*, filter_expression)` is parsed as `COUNT(filter_expression)`, and counting result is a number of records which 'filter_expression' IS NOT NULL. To avoid this situation, specify contant like '1'.

## Alerting

### Finding attackers

Query example for finding too many requests from single remote_host, to find DoS attacker:

```sql
SELECT
  remote_host,
  COUNT(*) as requests
FROM accesslog.win:time_batch(1 min)
GROUP BY remote_host
HAVING COUNT(*) >= 1000
```

Or to find brute force attacker:

```sql
SELECT
  remote_host,
  COUNT(*) AS failures
FROM authlog.win:time_batch(1 min)
WHERE result = 'failed'
GROUP BY remote_host
HAVING COUNT(*) >= 60
```

These queries are very simple, and we can fix threasholds or other conditions anytime.

### Summarize application logs

To tell application error logs to programmers, but not to flood notification center by one kind of messages:

```sql
SELECT
  level,
  file, line,
  LAST(host) AS host,
  LAST(message) AS message,
  COUNT(*) AS count
FROM application_logs.win:time_batch(5 min)
WHERE level IN ('error', 'critical')
GROUP BY level, file, line
```

Logs from a same point of code (same file, line and log-level) are expected to be a same log. Programmers can understand its severity by level and count, and get a first investigation step at a grance by sample of log message by `LAST(message)` in output events.
