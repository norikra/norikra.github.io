---
layout: layout
title: Norikra
subtitle: Schema-less Stream Processing with SQL
---
# Norikra - Schema-less Stream Processing with SQL
Norikra is a open source server software provides "Stream Processing" with SQL, written in JRuby, runs on JVM, licensed under GPLv2.

### Schema-less event streams (called as 'target')
Input/Output event streams as JSON objects, which can contain any fields with a target name.

### SQL processing
Norikra's query is SQL with window specifier support (It's actually Esper's EPL). JOINs and SubQueries are also fully supported.

### Complex events
Nested hashes and arrays are available for event values, and directly available from SQL queries.

### Dynamic query registration/removing
No more restarts required for new processing logs, queries, targets and any others.

### Ultra fast bootstrap, small start
Only 3 minutes required to install, configure and start norikra server.

### UDF plugins
UDFs can be added as plugins, written by yourself, or installed from RubyGems.org.

### Open source software
Norikra is licensed under GPLv2, and developed in github. Its core engine is based on Esper, which is GPLv2 library for SQL-based event processing.
 * https://github.com/norikra/norikra

Norikra is always open for users feedbacks and pull-requests.

## How to install and start
1. Install JRuby on your machine, and export PATH to jruby.
  * check `ruby -v` says like 'jruby 1.7.4' or same
2. Install norikra gem: `gem install norikra`
3. Start norikra: `norikra start`

WebUI is available on `http://hostname:26578/`.

* To daemonize
 * just do `norikra start --daemonize --logdir=/var/log/norikra`
 * with permission for pidfile /var/run/norikra/norikra.pid, and logdir
* And to stop
 * do `norikra stop`

Other options like supervisord, Upstart, systemd and others are available.

## Examples

Stay tuned!

## Software design

Coming soon.

## How to add UDF

Coming soon.

## FAQs

### Performance?

Later.

### Availability?

Later.

### vs Storm? vs Hive? vs RDBMS?

Later.

### How to pronounce 'Norikra'?

"No-rick-la"

### What 'Norikra' means?

Japanese famous mountains, which has great winding roads like dynamic data streams

## Test

hoge

## LICENSE
Norikra is licensed under GPLv2.
http://opensource.org/licenses/GPL-2.0
