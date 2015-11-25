# Welcome to mcrouter
Mcrouter is a memcached protocol router for scaling [memcached](http://memcached.org/) deployments. It's a core component of cache
infrastructure at Facebook and Instagram where mcrouter handles almost
5 billion requests per second at peak. See [FB engineering blog post about mcrouter](https://code.facebook.com/posts/296442737213493/introducing-mcrouter-a-memcached-protocol-router-for-scaling-memcached-deployments/).

Mcrouter is developed and maintained by Facebook.

Because the routing and feature logic are abstracted from the client in mcrouter deployments, the client may simply communicate with destination hosts through mcrouter over a TCP connection using standard memcached protocol. Typically, little or no client modification is needed to use mcrouter, which was designed to be a drop-in proxy between the client and memcached hosts.

Mcrouter supports typical memcache protocol commands like `get`, `set`, `delete`, etc. and specific commands to access stats, version and so on.

## Features 
+ [[Memcached ASCII protocol|Features#ascii-protocol]]
+ [[Connection pooling|Features#connection-pooling]]
+ [[Multiple hashing schemes|Pools#hash-functions]]
+ [[Prefix routing|Prefix routing setup]]
+ [[Replicated pools|Replicated-pools-setup]]
+ [[Production traffic shadowing|Shadowing-setup]]
+ [[Online reconfiguration|Command-line-options#config-management]]
+ [[Flexible routing|Config-Files]]
+ [[Destination health monitoring/automatic failover|Features#health-checkauto-failover]]
+ [[Cold cache warm up|Cold-cache-warm-up-setup]]
+ [[Broadcast operations|Multi-cluster-broadcast-setup]]
+ [[Reliable delete stream|Features#reliable-delete-stream]]
+ [[Multi-cluster support|Multi-cluster-broadcast-setup]]
+ Rich [[stats counters|Stats list]], [[Stats commands]] and [[debug commands|Admin requests]]
+ [[Quality of service|Features#quality-of-service]]
+ [[Large values|Features#large-values]]
+ [[Multi-level caches|Two-level-caching]]
+ [[IPv6 support|Features#ipv6-support]]
+ [[SSL support|Features#ssl-support]]

## News 
 * Release v0.14.0.  (November 25, 2015) Starting today, we will sync all mcrouter commits continuously to the master branch of our GitHub repo. In addition, we will be cutting a biweekly release branch that is intended to be stable and resilient to third-party incompatibilities. This week's release appears as the release-14-0 branch in our GitHub repo. 
 * Initial open source release (mcrouter 1.0) (September 15, 2014) 

## Getting Started
See [[installation|mcrouter-installation]] for more detailed installation instructions.

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
Copyright (c) 2015, Facebook, Inc. All rights reserved.

Licensed under a BSD license: https://github.com/facebook/mcrouter/blob/master/LICENSE