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
Mcrouter supports SSL encryption of incoming and/or outgoing connections. See [[SSL options|Command line options#ssl]]. This obviously needs support on the client that will talk to mcrouter and/or the destination (memcached). But note that two mcrouters set up in series can talk over SSL out of the box.

### Reliable delete stream
By default, mcrouter will save a record of every `delete` command it fails to deliver downstream (either due network issues or an error response from a destination). The client never gets an error response. `delete`s that were logged to disk are replied as `NOT_FOUND` to client; in this case mcrouter ensures that write to disk completes fully before sending the reply. The delete log can be replayed and cleaned up by another process.

See the [[delete stream options|Command line options#delete-stream]] for configuration details.

Number of failed deletes is recorded in the [[asynclog_requests stats counter|Stats-list#stats-logged-to-file]].

The failed delete log is called "asynclog" or "async spool". The log is written into files under the async spool root, organized into hourly directories. Each directory will contain multiple spool files (one per mcrouter process per thread per 15 minutes of log). Mcrouter never deletes or modifies those files in any way after writing to them.

An asynclog file is a sequence of '\n'-separated JSON serialized arrays. There are two asynclog formats: v1 and v2. v1 is being phased out and v2 will be the default format in the future.

##### Asynclog v1
A sample entry and explanation:
```
["AS1.0",1410611165.754,"C",["127.0.0.1",5001,"delete key\r\n"]]

[version (always "AS1.0"), timestamp, command (always "C"),
  [destination host, destination port, failed command]]
```

##### Asynclog v2
Version 2 has extra configuration information (pool name and mcrouter instance name) to route the failed request correctly, even if the destination host has changed. Destination host:port is still provided, but instance/pool data should be used to replay the delete if available.

A sample entry and explanation:

`["AS2.0",1410611229.747,"C",{"k":"key","p":"A","h":"[127.0.0.1]:5001","f":"5000"}]`

```
[version (always "AS2.0"), timestamp, command (always "C"),
  {"k": key,
   "p": pool name,
   "h": "[destination host]:destination port",
   "f": "mcrouter port or instance name"}]
```
