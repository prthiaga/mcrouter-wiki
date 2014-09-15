Mcrouter creates and automatically updates several files useful to monitor its state. These files are created under `/var/mcrouter/stats` folder by default and have names of form `libmcrouter.mcrouter.<mcrouter port number>.<name>`. Root folder for stats files is configured with [`--stats-root`](Command-line-options#--stats-root) command line option. These files by default are updated every 10 seconds, this period is configured with [`--stats-logging-interval`](Command-line-options#--stats-logging-interval) command line option.
Lets discuss each file in more details.

###startup_options
This file contains command line arguments passed to mcrouter. In case an option wasn't explicitly passed to mcrouter via command line, the default value will be listed here. More about command line options find [here](Command-line-options). Example for setup from [Home page](Home):

```JavaScript
 {
   "probe_delay_max_ms" : "60000",
   "probe_delay_initial_ms" : "10000",
   "cross_region_timeout_ms" : "0",
   "disable_tko_tracking" : "0",
   "config_str" : "{\"pools\":{\"A\":{\"servers\":[\"[::1]:5001\"]}},\"route\":\"PoolRoute|A\"}",
   "server_timeout_ms" : "1000",
   "disable_reload_configs" : "0",
   /* some options are omitted */
}
```

###config_sources_info
Contains [MD5](http://en.wikipedia.org/wiki/MD5) hashes for each tracked config file (additional files might be tracked due to `@import` statements). It provides the same information as [`__mcrouter__.config_sources_info`](Admin-requests) admin request.

###stats
Numerous counters collected by mcrouter. These include number of requests received and sent per operation, start time, average request duration, etc. For a detailed explanation on stats exposed by mcrouter see [stats logged to file](Stats-list#stats-logged-to-file). Example for setup from [Home page](Home) after running for 10 seconds:

```JavaScript
 {
   "libmcrouter.mcrouter.5000.config_last_success" : 1410744409,
   "libmcrouter.mcrouter.5000.uptime" : 10,
   "libmcrouter.mcrouter.5000.result_busy_shadow" : 0,
   "libmcrouter.mcrouter.5000.result_busy_failover" : 0,
   "libmcrouter.mcrouter.5000.cmd_get_out_failover" : 0,
   "libmcrouter.mcrouter.5000.cmd_meta" : 0,
   "libmcrouter.mcrouter.5000.result_busy_failover_count" : 0,
   "libmcrouter.mcrouter.5000.cmd_delete_out" : 0,
   "libmcrouter.mcrouter.5000.result_error" : 0
   /* some stats are omitted */
 }
```
