When data is spread across multiple clusters and datacenters, it is useful to group [pools](Pools) of cache hosts into logical groups and send requests to any particular cluster or datacenter.

When number of servers grows to thousands, it is useful to have a dedicated mcrouter instance per cluster and/or datacenter that serves requests from other clusters/datacenters. This way we can decrease number of connections to cache boxes, easily replicate data over clusters and datacenters, reduce load on clients.

This is why routing prefixes where introduced in mcrouter. Here is a configuration with two pools in different datacenters, each has own set of routing prefixes (or aliases), so clients can easily send requests to a particular pool. Also, to send a request to all clusters/datacenters, one can use [patterns](Routing-Prefix#pattern-matching) in key's routing prefix.

```JavaScript
{
  "pools": {
    "A": {
      "servers": [ /* hosts of pool A */ ]
    },
    "B": {
      "servers": [ /* hosts of pool B */ ]
    }
  },
  "routes": [
    {
      "aliases": [
        "/a/a/",
        "/A/A/"
      ],
      "route": "PoolRoute|A"
    },
    {
      "aliases": [
        "/b/b/"
      ],
      "route": "PoolRoute|B"
    }
  ]
}
```

_Explanation_: routing prefixes allow organizing pools into distinct clusters, and multiple clusters into datacenters. More about routing prefix concept [here](Routing-Prefix). In this example, commands sent to mcrouter "get /a/a/key" and "get /A/A/other_key" are served by servers in pool A (as "get key" and "get other_key" respectively), while "get /b/b/yet_another_key" will be served by servers in pool B (which will see "get yet_another_key").

More about mcrouter configuration see [here](Configuration).
