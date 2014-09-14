Mcrouter creates and automatically updates several files useful to monitor its state. These files are created under `/var/mcrouter/stats` folder by default and have names starting from `libmcrouter.mcrouter.<mcrouter port number>.<file>`. Root folder for stats files is configured with [`--stats-root`](Command-line-options#--stats-root) command line option. These files by default are updated every 10 seconds, this period is configured with [`--stats-logging-interval`](Command-line-options#--stats-logging-interval) command line option.
Lets discuss each file in more details.

###startup_options
This file contains command line arguments passed to mcrouter. In case an option wasn't explicitly passed to mcrouter via command line, the default value will be listed here. More about command line options find [here](Command-line-options).

###config_sources_info
[MD5](http://en.wikipedia.org/wiki/MD5) hashes for each tracked config file (additional files might be tracked due to `@import` statements).

###stats
Numerous counters collected by mcrouter. These include number of requests received and sent per operation, start time, average request duration, etc. For a detailed explanation on stats exposed by mcrouter see [Stats list](Stats-list)