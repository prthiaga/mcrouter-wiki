# Welcome to mcrouter
Mcrouter is a memcached protocol router for scaling [memcached](http://memcached.org/) deployments. It's a core component of cache
infrastructure at Facebook and Instagram where mcrouter handles almost
5 billion requests per second at peak.

Mcrouter is developed and maintained by Facebook.

Because the routing and feature logic are abstracted from the client in mcrouter deployments, the client may simply communicate with destination hosts through mcrouter over a TCP connection using standard memcached protocol. Typically, little or no client modification is needed to use mcrouter, which was designed to be a drop-in proxy between the client and memcached hosts.

Mcrouter supports typical memcache protocol commands like `get`, `set`, `delete`, etc. and specific commands to access stats, version and so on.

## Features 
+ [[Memcached ASCII protocol|Features#ascii-protocol]]
+ [[Connection pooling|Features#connection-pooling]]
+ Multiple hashing schemes
+ [[Prefix routing|Prefix routing setup]]
+ Replicated pools
+ Production traffic shadowing
+ Online reconfiguration
+ [[Flexible routing|Configuration]]
+ [[Destination health monitoring/automatic failover|Features#health-checkauto-failover]]
+ Cold cache warm up
+ Broadcast operations
+ [[Reliable delete stream|Features#reliable-delete-stream]]
+ Multi-cluster support
+ Rich [[stats counters|Stats list]], [[Stats commands]] and [[debug commands|Admin requests]]
+ [[Quality of service|Features#quality-of-service]]
+ [[Large values|Features#large-values]]
+ Multi-level caches
+ [[IPv6 support|Features#ipv6-support]]
+ [[SSL support|Features#ssl-support]]

## News 
 * Initial open source release (mcrouter 1.0) (September 14, 2014) 

## Getting Started
See [[installation|mcrouter-installation]] for more detailed installation instructions.

Mcrouter depends on [folly](https://github.com/facebook/folly) and [FBThrift](https://github.com/facebook/fbthrift).

The installation is a standard autotools flow:

```Shell
autoreconf --install
./configure
make
sudo make install
mcrouter --help
```

Assuming you have a memcached instance on the local host running on port 5001, the simplest mcrouter setup is

```Shell
mcrouter \
    --config-str='{"pools":{"A":{"servers":["127.0.0.1:5001"]}},"route":"PoolRoute|A"}' \
    -p 5000
```

To test, send a request to port 5000. For example, using [Netcat](http://netcat.sourceforge.net/):

```Shell
echo -ne "get key\r\n" | nc 0 5000
```

For a complete [[list of command line arguments|Command line options]], check `mcrouter --help`.

## Links
Engineering discussions and support: https://www.facebook.com/groups/mcrouter

## License
Copyright (c) 2014, Facebook, Inc. All rights reserved.

Licensed under a BSD license: https://github.com/facebook/mcrouter/blob/master/LICENSE
