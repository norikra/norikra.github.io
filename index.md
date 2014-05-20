---
layout: layout
title: Norikra
subtitle: Schema-less Stream Processing with SQL
---
# Schema-less Stream Processing with SQL
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

Query `www.search` matches the last event automatically.

For more query details and examples, see [Query of Norikra](/query.html) and [Query Examples](/examples.html).

### JSON Events over HTTP

Events (and all Norikra operations) can be passed by JSON over HTTP. HTTP JSON RPC port is 26578 in default (same as `--ui-port`).

Sending events with curl:

    curl -X POST -H "Content-Type: application/json" --data '{"target":"TARGETNAME", "events":[{event}, {event}]}' http://localhost:26578/api/send

And to fetch query output:

    # see (without removing)
    curl -X GET -H "Content-Type: application/json" --data '{"query_name":"QUERYNAME"}' http://localhost:26578/api/see
    # fetch
    curl -X POST -H "Content-Type: application/json" --data '{"query_name":"QUERYNAME"}' http://localhost:26578/api/event

## How to add UDF

UDFs/UDAFs can be loaded as plugin gems over rubygems or as private plugins.
In fact, Norikra's UDFs/UDAFs are Esper's plugin with a JRuby class to indicate plugin metadata.

Norikra's UDFs/UDAFs can be written in JRuby, or Java.

For details how to write your own UDF/UDAF for norikra and to release it as gem, see README of `norikra-udf-mock`.

https://github.com/norikra/norikra-udf-mock

## FAQs

### Client libraries?

* Ruby/JRuby
  * `gem install norikra-client` or `gem norikra-client-jruby`
  * https://github.com/norikra/norikra-client-ruby
* Perl
  * `cpanm Norikra::Client`
  * https://github.com/norikra/norikra-client-perl
* Python
  * https://github.com/norikra/norikra-client-python

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

Main performance arguments are CPUs and memories. Use options of `norikra start`: threading options and JVM options (see: [Norikra CLI](/cli-norikra.html#performance))

### Availability?

None. Duplicate streams and queries before Norikra server.

(Feature requests for high availability exists in TODOs)

### vs Storm? (or Kafka or ...)

Storm and other stream processing frameworks are framework, not processor. Storm has many features to distribute data and processing, but you should write processing code yourself, and deploy it, and then you should restart your distributed application.

Norikra is a stream processor. On Norikra, you should write queries only without any restarts. But norikra doesn't have features for distribution of data and processing.

If your service's event stream have huge traffic (over Gbps, over 1M events per seconds), Norikra does not fit for your problems. Storm or other stream processing frameworks work fine for such cases.

If there are many small event streams and many fragile queries, Norikra works fine.

Furthermore, Norikra's extremely fast bootstrapping helps you to build stream processing PoC or first version of your stream processing system. You can replace it with Storm or others when your event streams become bigger than norikra can handle it.

### vs Hive? vs Impala/Presto? vs RDBMS?

On Hive, Impala, Presto or RDBMS, we should manage when queries should be run, when these queries finished or where these results exists. For results every 5 minutes, we should maintain external system to kick queries per 5 minutes. Many queries at just a same time make troubles by disk I/O performance, swapped memories and high load averages.

On Norikra, once query registered, that runs forever. Problems like thundering herd does not occur because norikra's stream queries process events incrementally, and norikra does not use disks. By this reason, norikra is free from troubles of large size HDDs.

Splitting event streams is good idea. One is processed on Norikra, and the other is stored on storages, and processed by query engines like Hive/Impala/Presto or RDBMS. Both processing will be written by SQL... it is a variation of "Lambda architecture". We can get result as streams from norikra, and also get as hourly/daily or backup batches from query engines.

### Stream connectors?

Fluentd and fluent-plugin-norikra available.

 * http://fluentd.org/
 * https://github.com/norikra/fluent-plugin-norikra

Fluentd make log collection and delivery extremely easy, and have many plugins to store it on storages or to put it into tools/services for visualizations and notifications.

Norikra's stream input is very easy to connect fluentd's output, and Norikra's output is also easy to connect fluentd's input. So we can use Norikra as stream processor on Fluentd's data stream infrastructure.

<iframe src="http://www.slideshare.net/slideshow/embed_code/29174189" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px 1px 0; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/tagomoris/fluentpluginnorikra-fluentdcasual" title="fluent-plugin-norikra #fluentdcasual" target="_blank">fluent-plugin-norikra #fluentdcasual</a> </strong> from <strong><a href="http://www.slideshare.net/tagomoris" target="_blank">SATOSHI TAGOMORI</a></strong> </div>

Of course, other connectors contributions are welcome. Please send messages to @tagomoris on Twitter to do it.

### How to pronounce 'Norikra'?

"No-rick-la"

### What 'Norikra' means?

Japanese famous mountains, which has great winding roads like dynamic data streams

## LICENSE
Norikra is licensed under GPLv2.
http://opensource.org/licenses/GPL-2.0

### Logo
* designed by designer.du@hotmail.com
* Logo images are licensed under CC BY
