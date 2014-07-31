Routing prefixes allow organizing memcached pools into distinct clusters, and multiple clusters into datacenters. At the same time configuration remains simple - a common config file can be used in all clusters, and special syntax allows to make requests across clusters (or replicate to multiple clusters).

Let's set up a simple example. Say we have two clusters (a cluster contains both a set of clients that will talk to mcrouter and a set of memcached instances), and a third pool of memcached instances that we want to share between clusters. We want to set it up so that requests like `get my_key` from each cluster would go to memcached boxes in that cluster only, and requests like `get shr:my_key` with a special prefix `shr` (Note: there's nothing special about character `:` - it's just a convention we use) would go to the shared memcached pool. (TODO: add a picture).

It's possible to do all that with a single Mcrouter config - the only difference is the mcrouter command line in two clusters.

1. First cluster: `./mcrouter -f $CONFIG_FILE -p $PORT -R /datacenter/cluster0/`
2. Second cluster: `./mcrouter -f $CONFIG_FILE -p $PORT -R /datacenter/cluster1/`.

The mcrouter config looks like

```javascript
{
  "pools": {
    /* The elaborate names here are to highlight that mcrouter doesn't really care what they are -
       you will probably want to name them something more compact like "cluster0.local" */
    "local_pool_in_first_cluster": {
      "servers": [ /* hosts of pool */ ]
    },
    "local_pool_in_second_cluster": {
      "servers": [ /* hosts of pool */ ]
    },
    "shared_pool": {
      "servers": [ /* hosts of pool */ ]
    }
  },
  "routes": [
    {
      "aliases": [
        "/datacenter/cluster0/"
      ],
      "route": {
        "type": "PrefixSelectorRoute",
        "policies": {
          "shr": "PoolRoute|shared_pool"
        },
        "wildcard": "PoolRoute|local_pool_in_first_cluster"
      }
    },
    {
      "aliases": [
        "/datacenter/cluster1/"
      ],
      "route": {
        "type": "PrefixSelectorRoute",
        "policies": {
          "shr": "PoolRoute|shared_pool"
        },
        "wildcard": "PoolRoute|local_pool_in_second_cluster"
      }
    }
  ]
}
```

Ok, so what's going on here?  A _routing prefix_ is the string of the form `/a/b/` (allowed characters for `a` and `b` are `[A-Za-z0-9_-]`).  `-R` is the required command line parameter specifying the _default routing prefix_.  Every key in a request sent to mcrouter will contain either an _implicit_ or _explicit_ routing prefix.  If a key is already of the form `/a/b/key`, it has an explicit routing prefix `/a/b/`.  Otherwise, it has the implicit routing prefix that's the same as the default routing prefix specified on the command line.

A request will be routed according to the `aliases` list in the config. Let's work through some examples. Assume we're running in cluster 0 (i.e. mcrouter was started with `-R /datacenter/cluster0/`). (BTW, in case you're wondering the `datacenter` part is to allow grouping of clusters into multiple datacenters). Consider the following requests sent to mcrouter:

1. `get a`: there's no explicit prefix, so the key is processed as if it had the default prefix `/datacenter/cluster0/`.  The first section of the config matches; since `a` doesn't start with `shr` we end up in `wildcard` route and we route to `local_pool_in_first_cluster`.
2. `get /datacenter/cluster0/a`: there's an explicit prefix `/datacenter/cluster0/`. Since it's the same as the default prefix, the result is exactly the same as in first example.  Note: the prefix is stripped before the request is forwarded to memcached, so memcached box sees `get a` (this can be turned off as a pool setting, which can be useful for a setup where multiple mcrouters are connected in series).
3. `get /datacenter/cluster1/a`: goes to `local_pool_in_second_cluster`, even though we're running in first cluster.
4. `get shr:a`: goes to first part of the config, but prefix selector `shr` routes us to the `shared_pool`.
5. `get /datacenter/cluster1/shr:a`: goes to second part of the config, but prefix selector `shr` again routes us to the `shared_pool`, so exactly the same result as number 4.  This demonstrates that `get shr:a` in the second cluster will be delivered to the same shared pool.


### Pattern matching
You can also specify a _prefix pattern_ as part of the routing prefix.  Pattern matching supports single special character '*' which means 'any number of `[A-Za-z0-9_-]` characters'.  Examples:

1. `set /datacenter/*/a`: will resolve as two requests, `set /datacenter/cluster0/a` and `set /datacenter/cluster1/a`, so the same value will be set into both clusters.
2. `set /*/cluster*/a`: same result.

### Advanced: using macros to reduce config duplication
The config file above has a lot of repetition in it. Repetition is usually bad - it increases file size, reduces readability, increases probability of mistakes - so mcrouter supports a way to reuse common parts of the config. Let's look at what this will look like, this time with macros:

```javascript
{
  "pools": {
    "cluster0.local_pool": {
      "servers": [ /* hosts of pool */ ]
    },
    "cluster1.local_pool": {
      "servers": [ /* hosts of pool */ ]
    },
    "shared_pool": {
      "servers": [ /* hosts of pool */ ]
    }
  },
  "macros": {
    "cluster_route": {
      "type": "macroDef",
      "params": [ "id" ],
      "result": {
        "aliases": [
          "/datacenter/cluster%id%/"
        ],
        "route": {
          "type": "PrefixSelectorRoute",
          "policies": {
            "shr": "PoolRoute|shared_pool"
          },
          "wildcard": "PoolRoute|cluster%id%.local_pool"
        }
      }
    }
  },
  "routes": [
    "@cluster_route(0)",
    "@cluster_route(1)"
  ]
}
```
Remember, macro expansion happens before the config is parsed by mcrouter, and is completely agnostic of mcrouter's config semantics.  The expanded config will look very similar to the no macro version above (the only difference is the pool names).
