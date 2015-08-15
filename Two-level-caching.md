In cases when shared memcached pool can't handle the load and/or has too high latency, it is useful to setup 2 level caching. For example, one may run service and memcached on the same box and have shared cache pool with much bigger capacity.

In this case the logic on get path is following: read from local memcached instance, on miss go to shared pool to fetch data. If data is present in the shared pool, set it back to local memcached.

That's exactly what [WarmUpRoute](List-of-Route-Handles#warmuproute) does.

There is still an open question: how do we set data to both shared and local instances and how do we invalidate cache?
The answer depends on the use case. The simplest approach is to store data in local memcached with small TTL (exptime). That's useful for services that may tolerate stale data for a short period of time. 

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

_Explanation_: All sets and deletes go both to the "shared" and "local" pool. Gets are attempted on the "local" pool and in case of a miss, data is fetched from the "shared" pool (where the request is likely to result in a cache hit). If "shared" returns an hit, the response is then forwarded to the client and an asynchronous request updates the value in the "local" route handle. All data is set in "local" pool with small TTL (exptime) of 10 seconds.


Another approach is to replicate sets and deletes to all memcached instances (both shared pool and every local instance). It is a more complicated approach and it is useful only when a) stale data is totally unacceptable b) amount of gets is much higher than amount of sets/deletes.

The config for this approach looks like this:

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

_Explanation_: All sets and deletes go both to the "shared" pool and all servers in "all_local" pool. Gets are attempted on the "local" pool and in case of a miss, on the "shared" pool.

Find out more about mcrouter configuration [here](Config-Files).