###What is a mcrouter config?

Mcrouter config files specify where and how mcrouter should route requests.

A mcrouter config is JSON with some extensions:

* C++-style comments are allowed (both `/* */` and `\\`)
* Macros are supported (see [JSONM](JSONM))

Otherwise the file must conform to [JSON](http://json.org/) - unfortunately this means no trailing commas in lists or objects.

The top-level configuration object contains two properties: `pools` (optional), and `routes`
or `route`.

`pools` specifies groups of destination addresses for mcrouter requests, i.e. memcached hosts or other mcrouter instances.
`routes` (or `route`) specifies special handling (e.g, failover rules, key prefix handling). Mcrouter supports dynamic reconfiguration so you don't need to restart mcrouter to apply config changes.


###Quick examples

Too boring. I want to use it right now!

Okay, here are some common use cases (mcrouter is capable of much more):

####Example: split load between several memcache boxes
```javascript
{
  "pools": {
    "A": {
      "servers": [
        // your target memcached boxes here, e.g.:
        "127.0.0.1:12345",
        "[::1]:12346"
      ]
    }
  },
  "route": "PoolRoute|A"
}
```
_Explanation_: requests will be routed to boxes based on a consistent hashing of keys. For more on consistent hashing, see [here](http://en.wikipedia.org/wiki/Consistent_hashing).

####Example: failover requests to several boxes
```javascript
{
  "pools": { /* same as before */ },
  "route": {
    "type": "PrefixPolicyRoute",
    "operation_policies": {
      "get": "FailoverRoute|Pool|A",
      "set": "AllSyncRoute|Pool|A",
      "delete": "AllSyncRoute|Pool|A"
    }
  }
}
```
_Explanation_: deletes and sets are sent to all hosts in pool A, gets are sent to the first host in the pool. If a get request fails, it is automatically sent (retried) to the second host in pool, then the third, and so on.

####Example: shadow production traffic to test hosts
```javascript
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
        "index_range": [0, 1],
        // shadow requests for 10% of keys based on key hash
        "key_fraction_range": [0, 0.1]
      }
    ]
  }
}
```
_Explanation_: all requests go to the 'production' pool; requests for 10% of keys sent to the first and second host are **also** sent to the 'test' pool.

####Example: send to different hosts based on routing prefix
```javascript
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


###Defining pools

The `pools` property is a dictionary with pool names as keys and pool objects as values. Each pool object contains an ordered list of destination servers, together with some additional optional properties. Some of the pool properties are:

* `servers` (required)
List of `"host:port"` strings. Note that IPv6 addresses must be specified in square brackets.
```javascript
  "servers": [ "127.0.0.1:12345", "[::1]:5000", "memcached123.somedomain:4000" ]
```
Defines the pool's destination servers.

* `protocol` (optional): `"ascii"` (default) or `"umbrella"`
Which protocol to use. Ascii is the text Memcache protocol, Umbrella is an out of order binary protocol in use at Facebook. (see [Routing](Routing.md)).

* `keep_routing_prefix` (optional): bool (default `false`)
If true, do not strip the routing prefix from keys when sending request to this pool. Useful for making a mcrouter talk to another mcrouter.


###Defining `routes`

####Route handles

Routes are composed of blocks called "route handles". Each route handle encapsulates some piece of routing logic, such as "send a request to a single destination host" or "provide failover."

A route handle receives a request from a parent route handle, processes it, and potentially sends it on to child route handles; it then processes the replies and responds back with its own reply to the original request.

In a given config, route handles form a directed acyclic graph with each route handle as a node. They're freely composeable and an arbitrary graph can be represented in the config.

####Representing route handles in JSON

`routes` property is a list of route handle trees and `aliases`. `aliases` is a list of routing prefixes used for a given route handle tree. For example:

```javascript
{
  "routes": [
    {
      "aliases": [
        "/regionA/clusterA/",
        "/regionA/clusterB/"
      ],
      "route" : {
        // route handle tree for regionA
      }
    },
    {
      // other route here
    }
  ]
}
```

In this example, all keys with the `/regionA/clusterA/` and `/regionA/clusterB/` routing prefixes are routed to the regionA route handle tree.

Use `route` instead of `routes` if routing prefixes are not used. Semantically, it is equivalent to specifying the default route (given as a command line parameter) as the single alias, i.e.

```javascript
{
  "route": /* some route handle tree */
}
```

is equivalent to
```javascript
{
  "routes": [
    {
      "aliases": [ /* default routing prefix specified on mcrouter command line */ ],
      "route": /* some route handle tree */
    }
  ]
}
```

Each route handle object has `type` and additional fields. Here is an example of route handle object:
```javascript
{
  "type" : "HashRoute",
  "children" : "Pool|MyPool",
  ... // additional options, e.g. hash function, salt, etc.
}
```

For simplicity, there's an equivalent short form which sets all options to default values:
```javascript
 "HashRoute|Pool|MyPool"
```
You can read this as 'Create HashRoute that will route to pool MyPool'.

See [List of Route Handles](List-of-Route-Handles) for a more detailed description of available routes.


####Prefix route selector

In addition to selection by routing prefix, mcrouter also allows selection by key prefix. For example, you might want to route all keys beginning with foo_ to pool A, and all keys starting with bar_ to pool B. Note that key prefixes are not stripped, unlike routing prefixes.

To enable this logic, the root of the route handle tree should be a special route handle with the type set to "PrefixSelectorRoute" and the properties as follows:

* `policies`
 Object with the string prefixes as keys and the route handles as values. If a key matches multiple prefixes, the longest prefix is selected.
* `wildcard`
 A default route handle object. If a key doesn't match any prefix in `policies`, requests will go here.

Example:
```javascript
{
  "pools": { /* define pool A, B and C */ }
  "routes": [
    {
      "aliases": [ "/a/a/" ],
      "route": {
        "type": "PrefixSelectorRoute",
        "policies": {
          "a": "PoolRoute|A",
          "ab": "PoolRoute|B"
        },
        "wildcard": "PoolRoute|C"
      }
    }
  ]
}
```
_Explanation_: requests with routing prefix "/a/a/" and key prefix "a" (but not "ab"!) will be sent to pool A, requests with routing prefix "/a/a/" and key prefix "ab" will be sent to pool B. Other requests with routing prefix "/a/a/" will be sent to pool C. So key "/a/a/abcd" will be sent to pool B (as "abcd"); "/a/a/acdc" to pool A (as "acdc"), "/a/a/b" to pool C (as "b").


####Named handles

You may wish to use the same route handle in different parts of the config. To avoid duplication, add `name` to route handle object and then refer to this route handle by its name. If two route handles have the same name, only the first one is parsed - the second one is treated as a duplicate and is not parsed (even if it has different properties - these differences would be silently ignored). Instead, the first route handle is substituted in its place.

Route handles may also be defined in the `named_handles` property of the config. Example:
```javascript
{
  "pools": { /* define pool A and pool B */ },
  "named_handles": [
    {
      "type": "PoolRoute",
      "name": "ratedA",
      "pool": "A",
      "rates": {
        "sets_rate" : 10
      }
    }
  ],
  "routes": [
    {
      "aliases": [ "/a/a/" ],
      "route": {
        "type": "FailoverRoute",
        "children": [
          "ratedA",
          {
            "type": "PoolRoute",
            "name": "ratedB",
            "pool": "B",
            "rates": {
              "sets_rate": 100
            }
          }
        ]
      }
    },
    {
      "aliases": [ "/b/b/" ],
      "route": {
        "type": "FailoverRoute",
        "children": [
          "ratedB",
          "ratedA"
        ]
      }
    }
  ]
}
```
In this example, we specify two pools with different rate limits. Requests with the "/a/a/" routing prefix will be routed to pool A and failover to pool B. Requests with the "/b/b/" routing prefix will be routed to pool B, and failover to pool A.

###JSONM
You can use macros to avoid repetition when writing large complicated configs. For more information about JSONM and macros, see [JSONM](JSONM).