Once your dataset becomes too big to fit on one memcached server, you want to split it across multiple servers. This is called a horizontal partition of data or simply [sharding](http://en.wikipedia.org/wiki/Shard_(database_architecture)).
Mcrouter supports this feature. It can send requests to different destinations based on key hash. This way different keys will be evenly distributed across different targets and same key will be served by the same destination.
Mcrouter uses this logic to send requests to a [pool](Pools) by default. So sample configuration of `mcrouter` is pretty straightforward: just route to some pool.

```JavaScript
 {
   "pools": {
     "A": {
       "servers": [
         // your destination memcached boxes here, e.g.:
         "127.0.0.1:12345",
         "[::1]:12346"
       ]
     }
   },
   "route": "PoolRoute|A"
 }
```

_Explanation_: requests will be routed to boxes based on a consistent hashing of keys. For more on consistent hashing, see [here](http://en.wikipedia.org/wiki/Consistent_hashing).

More about mcrouter configuration see [here](Configuration).