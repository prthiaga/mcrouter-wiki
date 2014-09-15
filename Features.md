### ASCII protocol
Mcrouter implements the standard [https://github.com/memcached/memcached/blob/master/doc/protocol.txt](memcached ASCII protocol).
### Connection pooling
Mcrouter will only open one TCP connection to a given destination per mcrouter thread, and any requests sent to that destination will share the same connection.

Connections are open lazily, on the first request that needs to be sent to a destination, and are persistent once open. A connection that wasn't used for a while will be closed. This timeout is controlled via the `--reset-inactive-connection-interval` [[command line option|Command-line-options#network]].

### Health check/auto failover
Mcrouter keeps track of each destination's health status. If we get a certain number of timeouts in a row, the destination is marked "TKO" ("technical knockout" since it's down for the round :)) and we start sending `version` command probes (known as "TKO probes") until one comes back successfully. Normal requests would be failed over to a backup destination immediately (without waiting for further timeouts). We also distinguish between "soft TKO" and "hard TKO" - "hard TKO" are due to failures that are unlikely to be resolved on the subsequent request, for example `ECONNREFUSED`. See [[TKO command line options|Command line options#health-check]].

### Quality of service
Mcrouter can be thought of as a series of queues. The client-facing server parses the requests which are then forwarded to the routing code, which finally invokes internal client code that sends out requests to destinations. Check out [[the command line options|Command line options#queueing]] that control queueing in various parts of the pipeline.

### Large values
Mcrouter can split and reassemble values which are normally too large to store in memcached directly. The split threshold can be set with `--big-value-split-threshold=<N>` [[command line option|Command line options#routing]]. This is implemented with a BigValue [[route handle|Configuration#route-handles]].

On a large request we split it up into smaller chunks which are set as separate requests; we also set an index key. The index key is the same as the original request key, while the chunk request keys are composed of the original key + chunk index + a random suffix. The random suffix is recorded in the index key.

In this scheme, invalidating the index key is sufficient to invalidate the entire large value; and a large value set is atomic - concurrent set is decided by the index key update.

### IPv6 support
Mcrouter supports IPv6 hosts out of the box. To specify an IPv6 address in a pool server list, enclose the host in square brackets, i.e. on a dual-stacked host (both IPv6 and IPv4 interfaces) `"[::1]:5000"` is equivalent to `"127.0.0.1:5000"` or `"localhost:5000"`.

### SSL support
Mcrouter supports SSL encryption of incoming and/or outgoing connections. See [[SSL options|Command line options#SSL]]. This obviously needs support on the client that will talk to mcrouter and/or the destination (memcached). But note that two mcrouters set up in series can talk over SSL out of the box.

### Reliable delete stream
Link from Stats files about spooling goes here.