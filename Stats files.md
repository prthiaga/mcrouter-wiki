Mcrouter creates and automatically updates several files useful to monitor its state. These files are created under `/var/mcrouter/stats` folder by default and have names starting from `libmcrouter.mcrouter.<mcrouter port number>.<file>`. Root folder for stats files is configured with [`--stats-root`](Command-line-options#--stats-root) command line option. These files by default are updated every 10 seconds, this period is configured with `--stats-logging-interval` command line option.
Lets discuss each file in more details.

###startup_options
This file contains JSON object with command line options used by mcrouter. In case an option wasn't explicitly passed to mcrouter via command line, the default value will be listed here. More about command line options find [here](Command-line-options).

###config_sources_info
md5 hashes for each tracked config file (additional files might be tracked due to `@import` statements).

###stats
Numerous counters collected by mcrouter. These include number of requests received and sent per operation, start time, average request duration, etc.
Here is an explanation of what most important stats mean:
* asynclog_requests
  Number of failed deletes written to spool file. More about spool file read [here](Features#reliable-delete-stream).
* closed_inactive_connections
  Number of connections closed due to inactivity. Once connection is not used for more than a minute,
  mcrouter will force-close it. The period of inactivity is configured with
  `--reset-inactive-connection-interval` command line option.
