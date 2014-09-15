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
### IPv6 support
### SSL support
### Reliable delete stream
Link from Stats files about spooling goes here.