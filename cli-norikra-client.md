---
layout: layout
title: Norikra Client CLI
subtitle: Command line options of norikra-client
---
# Norikra Client CLI options

```
Commands:
  norikra-client admin CMD ...ARGS   # norikra server administrations
  norikra-client event CMD ...ARGS   # send/fetch events
  norikra-client field CMD ...ARGS   # manage target field/datatype definitions
  norikra-client help [COMMAND]      # Describe available commands or one specific command
  norikra-client query CMD ...ARGS   # manage queries
  norikra-client target CMD ...ARGS  # manage targets
  
Options:
  [--host=HOST]  
                 # Default: localhost
  [--port=N]     
                 # Default: 26571
```

### Global options

* host
* port

## target

```
Commands:
  norikra-client target close TARGET                                                              # close existing target and all its queries
  norikra-client target help [COMMAND]                                                            # Describe subcommands or one specific subcommand
  norikra-client target list                                                                      # show list of targets
  norikra-client target modify TARGET BOOL_VALUE                                                  # modify target to do define fields automatically or not
  norikra-client target open TARGET [fieldname1:type1 [fieldname2:type2 [fieldname3:type3] ...]]  # create new target (and define its fields)
```

## field

```
Commands:
  norikra-client field add TARGET FIELDNAME TYPE  # reserve fieldname and its type of target
  norikra-client field help [COMMAND]             # Describe subcommands or one specific subcommand
  norikra-client field list TARGET                # show list of field definitions of specified target
```

## query

```
Commands:
  norikra-client query add QUERY_NAME QUERY_EXPRESSION  # register a query
  norikra-client query help [COMMAND]                   # Describe subcommands or one specific subcommand
  norikra-client query list                             # show list of queries
  norikra-client query remove QUERY_NAME                # deregister a query
```

## event

```
Commands:
  norikra-client event fetch QUERY_NAME          # fetch events from specified query
  norikra-client event help [COMMAND]            # Describe subcommands or one specific subcommand
  norikra-client event send TARGET               # send data into targets
  norikra-client event sweep [query_group_name]  # fetch all output events of all queries of default (or specified) query group
```

## Admin
```
Commands:
  norikra-client admin help [COMMAND]  # Describe subcommands or one specific subcommand
  norikra-client admin stats           # dump stats json: same with norikra server's --stats option
```
