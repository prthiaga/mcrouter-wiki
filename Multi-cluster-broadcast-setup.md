When data is spread across multiple physical locations, it is useful to group [pools](Pools) of cache hosts into logical groups and send requests to any particular group. We use grouping by clusters and datacenters. In fact 'cluster' or 'datacenter' may have any number of pools which differ from others in one way or another. The main idea is those boxes are different (e.g. it takes longer to send a request there) hence we need to have a different routing logic.

The same logic applies when number of servers in some cluster or datacenter grows to thousands and we want to have a dedicated mcrouter instance to serve requests from other clusters/datacenters. Clients in other clusters will now route requests to a single mcrouter instance instead of thousands of cache boxes. This decreases number of connections to cache boxes, provides easy way to replicate data over numerous boxes and reduce load on clients.

This is why routing prefixes were introduced in mcrouter. Here is a configuration with two pools in different datacenters, each has own set of routing prefixes (or aliases), so clients can easily send requests to a particular pool. Also, to send a request to all clusters/datacenters, one can use [patterns](Routing-Prefix#pattern-matching) in key's routing prefix, thus easily replicate data.

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

You can start mcrouter via the `-R` option to specify the default route, e.g.:  `./mcrouter -f $CONFIG_FILE -p $PORT -R /a/a/` .  More details in [Routing-Prefix](Routing-Prefix).

_Explanation_: routing prefixes allow organizing pools into distinct clusters, and multiple clusters into datacenters. More about routing prefix concept [here](Routing-Prefix). In this example, commands sent to mcrouter `get /a/a/key` and `get /A/A/other_key` are served by servers in pool A (as `get key` and `get other_key` respectively), while `get /b/b/yet_another_key` will be served by servers in pool B (which will see `get yet_another_key`). Broadcast request `delete /*/*/one_more_key` will be sent to both pool A and pool B (as `delete one_more_key`).

Find out more about mcrouter configuration [here](Config-Files).