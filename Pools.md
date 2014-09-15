Destination hosts are grouped into "pools". A pool is a basic building block of a routing config. At a minimum, a pool consists of an ordered list of destination hosts and a hash function.

For detailed configuration, see [[PoolRoute|List-of-Route-Handles#poolroute]].

### Hash functions
- "ch3" ("consistent hash, version 3") is the consistent hashing algorithm used by mcrouter. This is the default hash function for pools. The underlying algorithm is [furc_hash()](https://github.com/facebook/mcrouter/blob/master/mcrouter/lib/fbi/hash.c#L151). The [[routing part of the key|Key syntax]] is hashed and the request is set to the resulting index in the pool.
- "crc32" hash sends the request to host index crc32(routing_key) % pool_size.
- HostIdRoute doesn't look at the key at all, instead always sending to hostid % pool_size, where hostid is some unique index associated with mcrouter's _local_ host. As a result, a single client will always talk to the same HostIdRoute destination, but two different clients will likely talk to different destinations.

### Shadowing
Mcrouter supports [[shadowing production traffic|Shadowing-setup]]. Shadowing works on the pool level. A pool (called "production pool" in the following discussion) can have multiple shadow pools; each shadow is defined with an `index_range`, which is a continuous range of production host indices; and a `key_fraction_range` which is a subinterval of [0.0, 1.0] and used for selecting a fraction of the total key space. Only requests which would go to one of the production hosts in the `index_range` with keys which would hash in the `key_fraction_range` (the hash used for this purpose is [SpookyHashV2](http://burtleburtle.net/bob/hash/spooky.html)) are sent to both the production and the shadow hosts.
