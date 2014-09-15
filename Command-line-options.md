### Delete stream
By default, mcrouter will save a record of every `delete` command it fails to deliver downstream (either due network issues or an error response from a destination). The client never gets an error response (deletes that were logged to disk are replied as `NOT_FOUND` to client; in this case mcrouter ensures that write to disk completes fully before sending the reply). The delete log can be replayed and cleaned up by another process.
- `--asynclog-disable` Disable logging of failed deletes; errors will be returned to the client.
- `-a <PATH>`, `--async-dir=<PATH>` Location for the failed deletes log.
- `--use-asynclog-version2` Enable new format log (the new format logs more information per delete and will become the default in the future).

### Runtime
- `--num-proxies=<N>` Run with N threads. Typically one thread per core is a good rule of thumb.
- `--fibers-max-pool-size=<N>` Maximum number of preallocated free fibers to keep around per thread. Should be set to the expected number of outgoing requests in flight - higher numbers will increase performance on request bursts at the expense of constantly higher memory usage.
- `--fibers-stack-size=<N>` The amount of stack to allocate per fiber, in bytes. Should normally not be adjusted unless significant code changes or unusually complicated configs are needed.
- `--fibers-debug-record-stack-size` ***Debug only.*** Record exact amount of fibers stacks used in the `fibers_stack_high_watermark` stat counter, which is normally an estimate (location at a likely bottom point of the stack). This is expensive and only provided for debugging. Works by painting the stack with a known value and periodically looking for the amount of undisturbed free space at the end of the stack.

### Routing config management
- `-f <PATH>`, `--config-file=<PATH>` The config file to use. Mcrouter will keep track of updates to the file; restarting the process is not required.
- `--config-str=<JSON>` Configure from a string instead of from file.
- `--validate-config` Check config for syntax and logic errors and exit immediately with good (zero) or error (non-zero) status.
- `--disable-reload-configs` Turn off automatic monitoring of configs (normally mcrouter will track any changes with `inotify` and update the runtime config on the fly).
- `--runtime-vars-file=<PATH>` Path to the runtime variables file (currently used for dynamic shadowing configuration).
- `--file-observer-poll-period-ms=<N>` Sleep for this amount between polling `inotify` for updates on the tracked files.
- `--file-observer-sleep-before-update-ms=<N>` Sleep for this amount after an update occurred. This is a hack to avoid reading a partially written config. For example, some text editors save the file in stages, and `inotify` will alert us on the first save, so we wait for a bit to work around this.
- `--constantly-reload-configs` ***Debug only.*** Continuously re-parse and re-load the config. Used to exercise the config reload code (for bugs and performance).

### Queueing
	    --proxy-max-inflight-requests                If non-zero, sets the limit on maximum incoming requests that will be routed in parallel by each proxy thread.  Requests over limit will be queued up until the number of inflight requests drops. [default: 0]
	    --destination-rate-limiting                  If not enabled, ignore "rates" in pool configs.
	    --target-max-inflight-requests               Maximum inflight requests allowed per target per thread (0 means no throttling) [default: 0]
	    --target-max-pending-requests                Only active if target-max-inflight-requests is nonzero. Hard limit on the number of requests allowed in the queue per target per thread.  Requests that would exceed this limit are dropped immediately. [default: 100000]
	    --max-client-outstanding-reqs                Maximum requests outstanding per client (0 to disable) [default: (uint32_t)((1024 * 1024 * 100) / (3 * 1024))]
	    --reqs-per-read                              Adjusts server buffer size to process this many requests per read. Smaller values may improve latency. [default: 0]


### Network
	-K, --keepalive-count                            set TCP KEEPALIVE count, 0 to disable [default: 0]
	-i, --keepalive-interval                         set TCP KEEPALIVE interval parameter in seconds [default: 60]
	-I, --keepalive-idle                             set TCP KEEPALIVE idle parameter in seconds [default: 300]
	    --reset-inactive-connection-interval         Will close open connections without any activity after at most 2 * interval ms. If value is 0, connections won't be closed. [default: 60000]
	    --tcp-rto-min                                adjust the minimum TCP retransmit timeout (ms) to memcached [default: -1]
	    --no-network                                 Debug only. Return random generated replies, do not use network.
	    --pem-cert-path                              Path of pem-style certificate for ssl [default: ""]
	    --pem-key-path                               Path of pem-style key for ssl [default: ""]
	    --pem-ca-path                                Path of pem-style CA cert for ssl [default: ""]

### Misc
	    --big-value-split-threshold                  If 0, big value route handle is not part of route handle tree,else used as threshold for splitting big values internally [default: 0]


### Routing
	-R, --route-prefix                               default routing prefix (ex. /oregon/prn1c16/) [default: "/././"]
	    --disable-miss-on-get-errors                 Disable reporting get errors as misses
	    --send-invalid-route-to-default              Send request to default route if routing prefix is not present in config

### Health check
	    --global-tko-tracking                        If enabled, track TKO per-router instead of per-proxy. This will become the default after testing in production
	-r, --probe-timeout-initial                      TKO probe retry initial timeout in ms [default: 10000]
	    --probe-timeout-max                          TKO probe retry max timeout in ms [default: 60000]
	    --timeouts-until-tko                         Mark as TKO after this many failures [default: 3]
	    --maximum-soft-tkos                          The maximum number of machines we can mark TKO if they don't have a hard failure. [default: 40]
	    --latency-window-size                        The number of samples to track when computing moving average latency for a proxy destination. If 0, TKO decisions based on latency are disabled. [default: 0]
	    --latency-threshold-us                       The maximum average destination latency (in us) that is considered acceptable. Destinations above this threshold will begin recording soft failures. [default: 2000000]

### Timeouts
	-t, --server-timeout                             server timeout in ms (DEPRECATED try to use cluster-server-timeout and regional-server-timeout) [default: 1000]
	    --cluster-pools-timeout                      server timeout for cluster pools in ms. Default value 0 means using deprecated server-timeout value for the flag [default: 0]
	    --regional-pools-timeout                     server timeout for regional pools in ms. Default value 0 means using deprecated server-timeout value for the flag [default: 0]
	    --cross-region-timeout-ms                    Timeouts for talking to cross region pool. If specified (non 0) takes precedence over every other timeout. [default: 0]
	    --cross-cluster-timeout-ms                   Timeouts for talking to pools within same region but different cluster. If specified (non 0) takes precedence over every other timeout. [default: 0]
	    --within-cluster-timeout-ms                  Timeouts for talking to pools within same cluster. If specified (non 0) takes precedence over every other timeout. [default: 0]

### Stats
	    --stats-root                                 Root directory for stats files [default: "/var/mcrouter/stats"]
	    --stats-logging-interval                     Time in ms between stats reports, or 0 for no logging [default: 10000]
	    --logging-rtt-outlier-threshold-us           surpassing this threshold rtt time means we will log it as an outlier. 0 (the default) means that we will do no logging of outliers. [default: 0]
	    --stats-async-queue-length                   Asynchronous queue size for logging. [default: 50]
	    --track-open-fds                             Log number of file descriptors opened by mcrouter process. This might cause performance regression in case number of mcrouter clients is huge.

### Standalone mcrouter options
	-L, --log-path                                   Log file path [default: ""]
	-p, --port                                       Port(s) to listen on (comma separated) [default: ]
	    --ssl-port                                   SSL Port(s) to listen on (comma separated) [default: ]
	    --listen-sock-fd                             Listen socket to take over [default: -1]
	-P, --pid-file                                   PID file [default: ""]
	-b, --background                                 Run in background
	-m, --managed-mode                               Managed mode (auto restart on crash)
	-n, --connection-limit                           Connection limit [default: 65535]