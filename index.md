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
  * check `ruby -v` says like 'jruby 1.7.8' or same
2. Install norikra gem: `gem install norikra`
3. Start norikra in foreground: `norikra start`

WebUI is available on `http://hostname:26578/`.

Norikra server doesn't save targets/queries in default.
Specify `--stats STATS_FILE_PATH` option to save these runtime configuration automatically.

    norikra start --stats /path/to/data/norikra.stats.json

JVM options like `-Xmx` are available:

    norikra start -Xmx2g

To daemonize (with permission for pidfile /var/run/norikra/norikra.pid, and logdir):

    norikra start -Xmx2g --daemonize --logdir=/var/log/norikra
    norikra start -Xmx2g --daemonize --pidfile /var/run/norikra.pid --logdir=/var/log/norikra
    # To stop
    norikra stop

Other options to daemonize: supervisord, Upstart, systemd and others are available.

Performance options about threadings:

    norikra start --micro     # or --small, --middle, --large

For other options, see help(`norikra help start`) or reference page:

http://norikra.github.io/cli-norikra.html

## Examples of queries and events

To add queries and events, use `norikra-client` command, which included in `norikra-client` gem and `norikra-client-jruby` gem. `norikra-client-jruby` gem will be installed with `norikra`, but it requires JVM execution waits. Without JVM, use `norikra-client` gem with CRuby.

For example, think about event streams related with one web service (ex: 'www'). At first, define `target` with mandantory fields (in other words, minimal fields set for variations of 'www' events).

    norikra-client target open www path:string status:integer referer:string agent:string userid:integer
    norikra-client target list

Supported types are `string`, `boolean`, `integer`, `float` and `hash`, `array`.

You can register queries when you want.

    # norikra-client query add QUERY_NAME  QUERY_EXPRESSION
    norikra-client query add www.toppageviews 'SELECT count(*) AS cnt FROM www.win:time_batch(10 sec) WHERE path="/" AND status=200'

And send events into norikra (multi line events [json-per-line] and LTSV events are also allowed).

    echo '{"path":"/", "status":200, "referer":"", "agent":"MSIE", "userid":3}' | norikra-client event send www
    echo '{"path":"/login", "status":301, "referer":"/", "agent":"MSIE", "userid":3}' | norikra-client event send www
    echo '{"path":"/content", "status":200, "referer":"/login", "agent":"MSIE", "userid":3}' | norikra-client event send www
    echo '{"path":"/page/1", "status":200, "referer":"/content", "agent":"MSIE", "userid":3}' | norikra-client event send www

Finally, you can get query outputs:

    norikra-client event fetch www.toppageviews
	{"time":"2013/05/15 15:10:35","cnt":1}
	{"time":"2013/05/15 15:10:45","cnt":0}

You can just add queries with optional fields:

    norikra-client query add www.search 'SELECT count(*) AS cnt FROM www.win:time_batch(10 sec) WHERE path="/content" AND search_param.length() > 0'

And send more events:

    echo '{"path":"/", "status":200, "referer":"", "agent":"MSIE", "userid":3}' | norikra-client event send www
    echo '{"path":"/", "status":200, "referer":"", "agent":"Firefox", "userid":4}' | norikra-client event send www
    echo '{"path":"/content", "status":200, "referer":"/login", "agent":"MSIE", "userid":3}' | norikra-client event send www
    echo '{"path":"/content", "status":200, "referer":"/login", "agent":"Firefox", "userid":4, "search_param":"news worldwide"}' | norikra-client event send www

Query 'www.search' matches the last event automatically.

## How to add UDF

UDFs/UDAFs can be loaded as plugin gems over rubygems or as private plugins.
In fact, Norikra's UDFs/UDAFs are Esper's plugin with a JRuby class to indicate plugin metadata.

For details how to write your own UDF/UDAF for norikra and to release it as gem, see README of `norikra-udf-mock`.

https://github.com/norikra/norikra-udf-mock

## Software design

Coming soon.

## FAQs

### Performance?

Not tested in details yet. Norikra's performance and throughput are affected by:

 * number of targets
 * number of queries
 * how complex queries are
 * how complex UDFs are

Current Status:

 * 10 queries
 * 2,000 events per seconds
 * 5% usage of 4core CPU

Main performance arguments are CPUs and memories. Use options of `norikra start`:

 * threading options (see: http://norikra.github.io/cli-norikra.html#performance )
 * `-X` jvm options
   * memory options like `-Xmx`
   * GC options like `-XX:+UseConcMarkSweepGC`
   * and many many others

### Availability?

None. Duplicate streams and queries before Norikra server.

(Feature requests for high availability exists in TODOs)

### vs Storm? vs Hive? vs RDBMS?

TODO

### Stream connectors?

Fluentd and fluent-plugin-norikra available.

 * http://fluentd.org/
 * https://github.com/norikra/fluent-plugin-norikra

TODO: more details

### How to pronounce 'Norikra'?

"No-rick-la"

### What 'Norikra' means?

Japanese famous mountains, which has great winding roads like dynamic data streams

## LICENSE
Norikra is licensed under GPLv2.
http://opensource.org/licenses/GPL-2.0
