When different data is stored on the same memcached instance, problem of different workloads arises.

Memcached doesn't distinguish or prioritize any keys, all keys compete for the same amount of memory and are evicted in the same way. But some data is simple to compute, some is more complex; so cache miss on more 'expensive' data results in higher load on clients, while 'cheaper' data can be easily recalculated. Also some keys are requested more often than others, so data that is requested not-so-often can be evicted from cache even before any client uses it.

Mcrouter solves this problem by separating keys from different workloads to different pools. 

Mcrouter can send keys with different prefixes to different [pools](Pools), so they will not compete for the same memory space.

```JavaScript
 {
   "pools": {
     "workload1": { "servers": [ /* list of cache hosts for workload1 */ ] },
     "workload2": { "servers": [ /* list of cache hosts for workload2 */ ] },
     "common_cache": { "servers": [ /* list of cache hosts for common use */ ] }
   },
   "route": {
     "type": "PrefixSelectorRoute",
     "policies": {
       "a": "PoolRoute|workload1",
       "b": "PoolRoute|workload2"
     },
     "wildcard": "PoolRoute|common_cache"
   }
 }
```

_Explanation_: requests with key prefix "a" will be sent to pool 'workload1', requests with key prefix "b" will be sent to pool 'workload2'. Other requests will be sent to pool 'common_cache'. So key "abcd" will be sent to 'workload1'; "bar" to 'workload2'; "zzz" to 'common_cache'.

Find out more about mcrouter configuration [here](List-of-Route-Handles).