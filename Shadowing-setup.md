Even software engineers make mistakes. In fact, without proper testing any code change in critical code is dangerous and can bring down the entire site. Also some optimizations look great in theory but when tested under production load they doesn't show any benefit, or even break the existing logic. That's why testing under real load is the best way to find bugs in new code.

Mcrouter can send some part of production traffic (shadow it) to test hosts, without any visible impact on performance or reliability. Here is an example:

```JavaScript
{
  "pools": {
    "production": {
      "servers": [ /* production hosts */ ]
    },
    "test": {
      "servers": [ /* test hosts */ ]
    }
  },
  "route": {
    "type": "PoolRoute",
    "pool": "production",
    "shadows": [
      {
        "target": "PoolRoute|test",
        // shadow traffic that would go to first and second hosts in 'production' pool
        // note that the endpoint is non-inclusive
        "index_range": [0, 2],
        // shadow requests for 10% of keys based on key hash
        "key_fraction_range": [0, 0.1]
      }
    ]
  }
}
```

_Explanation_: all requests go to the 'production' [pool](Pools); requests for 10% of keys sent to the first and second production hosts are **also** sent to the 'test' pool.

Find out more about mcrouter configuration [here](Configuration).