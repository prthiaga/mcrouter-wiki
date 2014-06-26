Essentially, mcrouter takes in Memcache requests, applies rules defined by mcrouter configs, and sends them on. Configuration allows for flexible specification of special handling and failover behavior.

###Architecture 


###Typical Use Cases


###Command line arguments

Assuming you have a memcached instance on the local host running on port 5001, the simplest mcrouter setup is <code>(::1,/code>, which is the IPv6 loopback address. IPv6 addresses must be specified in square brackets. You may also use IPv4 addressing, e.g, "127.0.0.1:5001" or specify a port on localhost, e.g., <code>"localhost:5001"):</code>.

```Shell
./mcrouter --config-str='{"pools":{"A":{"servers":["[::1]:5001"]}},"route":"PoolRoute|A"}' -p 5000
```

To verify mcrouter works, send a request to port 5000. For example, using
Netcat (http://netcat.sourceforge.net/):

```Shell
echo -ne "get key\r\n" | nc 0 5000
```

For a complete list of command line arguments, see `./mcrouter --help`.

###Routing

Mcrouter supports typical Memcache protocol commands like get, set, delete, et cetera, and commands to access stats, version and so on. For more information on routing and other Mcrouter commands, see [here](Routing).

###Configuration

Mcrouter is configured with JSON files called mcrouter configs, which primarily consist of pool and route information.  See [Configuration](mcrouter-configuration) for information. 