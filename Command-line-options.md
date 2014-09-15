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
Mcrouter can be thought of as a series of queues. The client-facing server parses the requests which are then forwarded to the routing code, which finally invokes internal client code that sends out requests to destinations. These options control queueing in various parts of the pipeline, and are listed in the same order:
- `--reqs-per-read=<N>` If specified, the incoming request buffer size is automatically adjusted to allow through roughly this many requests per event loop iteration. Smaller values may improve latency in a large fan out scenario.
- `--max-client-outstanding-reqs=<N>` Maximum requests outstanding (that is, sent by the client but not yet replied by mcrouter) per client connection. Mcrouter stops reading from the connection socket when this limit is reached. This limits overall memory usage by mcrouter.
- `--proxy-max-inflight-requests=<N>` Maximum requests that were routed through the config but not replied yet. This is a queue between the incoming parsed requests and individual fibers allocated per request. The rationale is to limit the number of fibers in use, which require more memory than the parsed requests.
- `--destination-rate-limiting` Enable `rates` options in pool configurations, which allow to throttle requests to destination by operation type.
- `--target-max-inflight-requests=<N>` Maximum outstanding requests allowed to be sent per destination. Requests over this limit will queue up in mcrouter.
- `--target-max-pending-requests=<N>` The size of the outstanding requests queue per destination. Requests over this limit will be returned with an error immediately.


### Network
- `--reset-inactive-connection-interval=<N>` A timer with this period (in ms) closes all inactive outgoing connections. 0 to disable.
- `-K <N>`, `--keepalive-count=<N>` TCP KEEPALIVE count for outgoing connections.
- `-i <N>`, `--keepalive-interval=<N>` TCP KEEPALIVE interval for outgoing connections.
- `-I <N>`, `--keepalive-idle=<N>` TCP KEEPALIVE idle for outgoing connections.
- `--pem-cert-path=<PATH>`, `--pem-key-path=<PATH>`, `--pem-ca-path=<PATH>` Paths of pem-style certificate/key/CA cert for SSL (used for both incoming and outgoing connections).
- `--no-network` ***Debug only.*** Return random generated replies to every request, do not use network.

### Routing
- `-R <PREFIX>`, `--route-prefix=<PREFIX>` Default [[routing prefix]] (e.g. `/datacenter/cluster/`).
- `--disable-miss-on-get-errors` By default, mcrouter converts any errors on `get` commands into misses (i.e. `NOT_FOUND`) for the client. This option disables this behaviour, forwarding back all actual errors.
- `--send-invalid-route-to-default` Normally requests with keys of the form `/foo/bar/key` will result in an error if `/foo/bar/` does not explicitly exist in the config. With this option, mcrouter will send those requests to default route (respecting `keep_routing_prefix` pool option, and forwarding `/foo/bar/` unmodified if it's set).
- `--big-value-split-threshold=<N>` Values in update requests over this threshold in bytes will be automatically split into smaller values (re-assembly on `get` is also automatic). If 0, no splitting is performed.

### Health check
Mcrouter keeps track of each destination's health status. If we get a certain number of timeouts in a row, the destination is marked "TKO" ("technical knockout" since it's down for the round :)) and we start sending `version` command probes (known as "TKO probes") until one comes back successfully. Normal requests would be failed over to a backup destination immediately (without waiting for further timeouts). We also distinguish between "soft TKO" and "hard TKO" - "hard TKO" are due to failures that are unlikely to be resolved on the subsequent request, for example `ECONNREFUSED`.
- `--timeouts-until-tko=<N>` Mark as soft TKO after this many timeouts.
- `-r <N>`, `--probe-timeout-initial=<N>`; `--probe-timeout-max=<N>` TKO probes are sent with exponentially increasing interval. These options control the initial size of this interval and the maximum size respectively (in ms). The actual intervals have some random jitter of up to 50% added to them to avoid overloading a single failed host with TKO probes from different mcrouters.
- `--maximum-soft-tkos=<N>` Maximum number of destinations allowed to be in soft TKO state at any point in time. This is for cascading failure protection. We stop marking hosts as TKO on timeout once we reach this limit.
- `--latency-threshold-us=<N>` If nonzero, destinations with average latency longer than this threshold will have new requests registered as soft TKO events.
- `--latency-window-size=<N>` Destination latency moving average window size. Smaller numbers will put more weight on the new requests (i.e. at the extreme the value of 1 will cause the average to be equal to the latest request latency), while larger numbers put less weight on unusual bursts. If 0, TKO decisions based on latency are disabled.
- `--global-tko-tracking` If enabled, track TKO per-router instead of per-proxy. This will become the default after testing in production.

### Timeouts
The simplest way is to specify destination timeouts is with
- `-t <N>`, `--server-timeout=<N>` Destination timeout in ms.

There are two ways to fine tune timeouts in a geographically distributed setup. A crude way is to add a `pool_locality` setting to each pool (set to literal strings `cluster` or `region`) and specify
- `--cluster-pools-timeout=<N>` Destination timeout for cluster pools in ms.
- `--regional-pools-timeout=<N>` Destination timeout for regional pools in ms.

A better way is to specify `region` and `cluster` pool settings. These are compared to the components of the default route (i.e. `/some_region/some_cluster/`). Depending on whether we're in the same cluster as the pool, same region, or a different region entirely, one of the following timeouts is in effect:
- `--cross-region-timeout-ms=<N>` Timeout for talking to pools in another region.
- `--cross-cluster-timeout-ms=<N>` Timeout for talking to pools within the same region but a different cluster.
- `--within-cluster-timeout-ms=<N>` Timeout for talking to pools within the same cluster.

### Stats
- `--stats-root=<PATH>` Root directory for stats files.
- `--stats-logging-interval=<N>` Time in ms between stat file updates, or 0 for no logging.
- `--logging-rtt-outlier-threshold-us=<N>` Destination requests with round trip time exceeding this threshold will be considered "outliers" and counted in the outliers stats.
- `--track-open-fds` Log number of file descriptors opened by mcrouter process. This might cause performance regression if the number of connections is huge, as this is computed by walking the `/proc/self/fd` directory.

### Standalone mcrouter options
- `-L <PATH>`, `--log-path=<PATH>` Path for the log file.
- `-p <PORT1>,<PORT2>,...`, `--port <PORT1>,<PORT2>,...` Port(s) to listen on (comma separated).
- `--ssl-port <PORT1>,<PORT2>` SSL Port(s) to listen on (comma separated).
- `--listen-sock-fd` Listen socket fd to take over (used in tests).
- `-P <PATH>`, `--pid-file=<PATH>` If specified, open and lock the PID file to prevent another mcrouter start up with the same PID file path.
- `-b`, `--background` Daemonize on startup.
- `-m`, `--managed-mode` Spawn a child mcrouter; parent process is responsible for auto restarts.
- `-n <N>`, `--connection-limit=<N>` File descriptor limit.