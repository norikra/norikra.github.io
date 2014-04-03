---
layout: layout
title: Tips
subtitle: miscellaneous features
---
# Tips

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

Stats will be dumped with SIGUSR2 if `--stats PATH` specified. Or, in client side, `norikra-client admin stats` can export stats json data.
