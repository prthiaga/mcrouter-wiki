Every time a new cache box is added to Memcache infrastructure, it has empty cache. This box is called 'cold'. Every request to this box will end up in 'miss', so clients will spend lots of resources to fill the cache. In the worst case this will affect performance of the entire site, because on every 'miss' on the cold box client would spend much more time to actually get the data.

Mcrouter offers a way to 'warm up' cold cache boxes without impact on performance. The idea is simple: when we have a cold box and receive a 'miss', try to get a value from 'warm 'box (one with filled cache) and set it back to cold box.

Here is the setup for this logic:

```JavaScript
 {
   "pools": {
     "cold": { "servers": [ /* cold hosts */ ] },
     "warm": { "servers": [ /* warm hosts */ ] }
   },
   "route": {
     "type": "WarmUpRoute",
     "cold": "PoolRoute|cold",
     "warm": "PoolRoute|warm"
   }
 }
```

_Explanation_: All sets and deletes go to the "cold" route handle. Gets are attempted on the "cold" route handle and in case of a miss, data is fetched from the "warm" route handle (where the request is likely to result
in a cache hit). If "warm" returns an hit, the response is then forwarded to the client and an asynchronous request updates the value in the "cold" route handle.

Find out more about mcrouter configuration [here](Configuration).