---
layout: layout
title: Norikra CLI
subtitle: Command line options of norikra
---
# Norikra CLI options

```
Usage:
  norikra start [-Xxxx] [other options]
```

```
Options:
  -H, [--host=HOST]                   # host address that server listen [0.0.0.0]
  -P, [--port=N]                      # port that server uses [26571]
  -s, [--stats=STATS]                 # status file path to load/dump targets, queries and server configurations [none]
      [--suppress-dump-stat]          # specify not to update stat file with updated targets/queries/configurations on runtime [false]
      [--dump-stat-interval=N]        # interval(seconds) of status file dumps on runtime [none (on shutdown only)]
  -d, [--daemonize]                   # daemonize Norikra server [false (foreground)]
  -p, [--pidfile=PIDFILE]             # pidfile path when daemonized [/var/run/norikra/norikra.pid]
                                      # Default: /var/run/norikra/norikra.pid
      [--outfile=OUTFILE]             # stdout redirect file when daemonized [${logdir}/norikra.out]
      [--micro]                       # development or testing (inbound:0, outbound:0, route:0, timer:0, rpc:2)
      [--small]                       # virtual or small scale servers (inbound:1, outbount:1, route:1, timer:1, rpc:2)
      [--middle]                      # rackmount servers (inbound:4, outbound:2, route:2, timer:2, rpc:4)
      [--large]                       # high performance servers (inbound: 6, outbound: 6, route:4, timer:4, rpc: 8)
      [--inbound-threads=N]           # number of threads for inbound data
      [--outbound-threads=N]          # number of threads for outbound data
      [--route-threads=N]             # number of threads for events routing for query execution
      [--timer-threads=N]             # number of threads for internal timers for query execution
      [--inbound-thread-capacity=N]   
      [--outbound-thread-capacity=N]  
      [--route-thread-capacity=N]     
      [--timer-thread-capacity=N]     
      [--rpc-threads=N]               # number of threads for rpc handlers
      [--web-threads=N]               # number of threads for WebUI handlers
  -l, [--logdir=LOGDIR]               # directory path of logfiles when daemonized [nil (console for foreground)]
      [--log-filesize=LOG-FILESIZE]   # log rotation size [10MB]
      [--log-backups=N]               # log rotation backups [10]
      [--more-quiet]                  # set loglevel as ERROR
  -q, [--quiet]                       # set loglevel as WARN
  -v, [--verbose]                     # set loglevel as DEBUG
      [--more-verbose]                # set loglevel as TRACE
  -h, [--help]                        # show this message
```

```
Usage:
  norikra stop [options]
  
Options:
  -p, [--pidfile=PIDFILE]  # pidfile path when daemonized [/var/run/norikra/norikra.pid]
                           # Default: /var/run/norikra/norikra.pid
      [--timeout=N]        # timeout seconds to wait process exit [5]
                           # Default: 5
```

## <a name="performance"></a>Performance option details

Threads option available with `norikra start`. Simple specifiers for performance with threadings:

    norikra start --micro     # or --small, --middle, --large (default: 'micro')

Norikra server has 3 types of threads:

* engine: 4 query engine thread types on Esper
  * inbound: input data handler for queries
  * outbound: output data handler for queries
  * router: event handler which decides which query needs that events
  * timer: executer for queries with time_batches and other timing events
* rpc: data input/output rpc handler threads on Jetty
* web: web ui request handler threads on Jetty

In many cases, norikra server handling high rate events needs large number of rpc threads to handle input/output rpc requests. WebUI don't need threads rather than default in almost all of cases.

Engine threads depends on queries running on norikra, input/output event data rate and target numbers. For more details, see Esper's API Documents: http://esper.codehaus.org/esper-4.10.0/doc/reference/en-US/html/api.html#api-threading

Norikra's simple specifiers details of threadings are:

* micro: development and testing
  * engine: all processings on single threads
  * rpc: 2 threads
  * web: 2 threads
* small: low rate events on virtual servers
  * engine: inbound 1, outbound 1, route 1, timer 1 threads
  * rpc: 2 threads
  * web: 2 threads
* middle: high rate events on physical servers
  * engine: inbound 4, outbound 2, route 2, timer 2 threads
  * rpc: 4 threads
  * web: 2 threads
* large: inbound heavy traffic and huge amount of queries
  * engine: inbound 6, outbound 6, route 4, timer 4 threads
  * rpc: 8 threads
  * web: 2 threads

To specify sizes of each threads, use `--*-threads=NUM` options.

