When a single shared memcached pool is overloaded it could be useful to use a 2-level caching setup. For example, an additional memcached instance could be co-located with the client in addition to a shared cache pool with a much bigger capacity.

Then the `get` logic becomes:
- Read from the local memcached instance
- On a miss, read from the shared pool
- If the data was present in the shared pool, set it back to the local memcached

This is exactly what [WarmUpRoute](List-of-Route-Handles#warmuproute) does.

We also need to decide:
- How are updates handled - how do we set the data to both the shared and the local instances?
- How are deletes handled?

The answers depend on the use case. Some solutions are suggested below.


#### Local instance with small TTL

A simple approach is to use a small TTL (exptime) when setting to the local cache - assuming the services can tolerate stale data up to that period of time.

Here is the corresponding config:

```JavaScript
 {
   "pools": {
     "shared": { "servers": [ /* shared memcached hosts */ ] },
     "local": { "servers": [ /* local memcached instance (e.g. "localhost:<port>") */ ] }
   },
   "route": {
     "type": "OperationSelectorRoute",
     "operation_policies": {
       "get": {
         "type": "WarmUpRoute",
         "cold": "PoolRoute|local",
         "warm": "PoolRoute|shared",
         "exptime": 10
       }
     },
     "default_policy": {
       "type": "AllSyncRoute",
       "children": [
         "PoolRoute|shared",
         {
           "type": "ModifyExptimeRoute",
           "target": "PoolRoute|local",
           "exptime": 10
         }
       ]
     }
   }
 }
```

_Explanation_: All sets and deletes will go to both the `shared` and `local` pools. Gets only go to the `local` pool first; the `shared` pool will be checked in case of a miss. If `shared` returns a hit, the response is then returned to the client and an asynchronous request updates the value in the `local` pool - note that the client is not blocked on that update. All data is set in the `local` pool with a small TTL (exptime) of 10 seconds.


#### Replicate everything

Another approach is to replicate sets and deletes to all memcached instances (both the shared pool and any local instances). This is useful if stale data is unacceptable, but the tradeoff is that the update and delete traffic is increased N-fold (where N is the number of local instances) - don't use if your load is write-heavy!

The config for a single instance might look like this:

```JavaScript
 {
   "pools": {
     "shared": { "servers": [ /* shared memcached hosts */ ] },
     "local": { "servers": [ /* local memcached instance (e.g. "localhost:<port>") */ ] },
     "all_local": { "servers": [ /* ALL local memcached instances across the fleet */ ] }
   },
   "route": {
     "type": "OperationSelectorRoute",
     "operation_policies": {
       "get": {
         "type": "MissFailoverRoute",
         "children": [
           "PoolRoute|local",
           "PoolRoute|shared"
         ]
       }
     },
     "default_policy": {
       "type": "AllSyncRoute",
       "children": [
         "PoolRoute|shared",
         "Pool|all_local"
       ]
     }
   }
 }
```

_Explanation_: All sets and deletes go to both the `shared` pool and all local instances (`all_local` pool). Gets are attempted on the `local` pool and in case of a miss, on the `shared` pool.

Find out more about mcrouter configuration [here](Config-Files).