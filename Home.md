# Welcome to mcrouter
Mcrouter is a Memcache protocol routing layer that enables the easy expansion of memcached deployments to distributed pools of arbitrary side, providing request routing, connection pooling, failover, and many other features. Because the routing and feature logic are abstracted from the client in mcrouter deployments, the client may simply communicate with destination hosts through mcrouter over a TCP connection using standard Memcache protocol. 

Typically, little or no client modification is needed to use mcrouter, which was designed to be a drop-in proxy between the client and memcached hosts. At Facebook and Instagram, mcrouter is a core component of a distributed cache infrastructure that spans a very large number of individual memcached boxes.

Mcrouter supports typical memcache protocol commands like `get`, `set`, `delete`, etc. and specific commands to access stats, version and so on. See [Routing](Routing) for more.

For a detailed introduction to mcrouter, see [Overview](Overview). 


## Features 
 * Request routing
 * Connection Pooling 
 * Flexible Configuration 
 * Failover support
 * Shadow testing support


## News 
 * mcrouter v1.0 Released (September, 2014) 
 * etc. 

## Getting Started
To install Mcrouter, see [Installation](mcrouter-installation).

Assuming you have a memcached instance on the local host running on port 5001, the simplest Mcrouter setup is as following (Note, "::1" is the IPv6 loopback address; IPv6 addresses must be specified in square brackets. You can also use "127.0.0.1:5001" or "localhost:5001"):

```Shell
./mcrouter --config-str='{"pools":{"A":{"servers":["[::1]:5001"]}},"route":"PoolRoute|A"}' -p 5000
```

To test, send a request to port 5000. For example, using Netcat (http://netcat.sourceforge.net/):

```Shell
echo -ne "get key\r\n" | nc 0 5000
```

For a complete list of command line arguments, check `./mcrouter --help`.

## Links

## Contact