`get` requests with keys which start with a prefix `__mcrouter__` cause mcrouter to return special values and are not forwarded downstream.

***

To illustrate some of these commands we'll use the following unnecessarily convoluted config, with some frivolous macros and an `@import` thrown in for good measure:

Main config, `test.json`:
```
{
  "macros": {
    "myRoute": {
      "type": "macroDef",
      "params": ["poolName"],
      "result": "PoolRoute|%poolName%"
    }
  },
  "pools": "@import(pools.json)",
  "route": {
    "type": "AllSyncRoute",
    "children": [
      "@myRoute(foo)",
      "@myRoute(bar)"
    ]
  }
}
```

Extra file, `pools.json`:
```
{
  "foo": { "servers": ["localhost:10001"] },
  "bar": { "servers": ["localhost:10002"] }
}
```

Start mcrouter with `mcrouter -f config.json -p 5000`.

***

The following special commands are supported:

- `get __mcrouter__.version`: version string of the build (same string as returned by `version`).
- `get __mcrouter__.config`: config string (error if configured from a file).
- `get __mcrouter__.config_age`: how long, in seconds, since last config reload.
- `get __mcrouter__.config_file`: config file location (error if configured from string).
- `get __mcrouter__.options`: list of space-separated option name and value pairs of startup command line options.
- `get __mcrouter__.options(num_proxies)`: the value of a specific option, in this example `num_proxies`.
- `get __mcrouter__.route(<op>,<key>)`: list of destination where a given request would be routed to. Note that there are no spaces around comma or brackets, as the whole `__mcrouter__...` string must be a valid memcache key.

  Example: `get __mcrouter__.route(set,mykey)` =>
  
  ```
  VALUE __mcrouter__.route(set,mykey) 0 32
  localhost:10001
  localhost:10002
  END
  ```

  This means that `set key` would be sent to both `localhost:10001` and `localhost:10002`.  Note that this follows the exact routing logic in effect, i.e. if the main destination is down and there's a failover set up, `route` will report the failover destination.

- `get __mcrouter__.route_handles(<op>,<key>)`: representation of the route handle graph generated from the config that a given request would traverse.

  Example: `get __mcrouter__.route_handles(set,mykey)` =>

  ```
  VALUE __mcrouter__.route_handles(set,mykey) 0 190
  proxy
   root
    all-sync
     asynclog
      hash
       host|pool=foo|id=0|ssl=false|ap=localhost:10001:TCP:ascii
     asynclog
      hash
       host|pool=bar|id=0|ssl=false|ap=localhost:10002:TCP:ascii
  END
  ```
  
  Here we see some route handles directly from the config, like `all-sync`, and some that were generated internally - e.g. `PoolRoute` is actually converted into a combination of `asynclog` and `hash` route handles in this example.

  Note that leaf nodes also include some additional info for each destination.  `pool` specifies the name of the pool the host belongs to; `id` is the index of the destination in the pool's server list; `ssl` is true iff SSL is enabled for the destination; finally `ap` is the accesspoint string as specified in the config (in this case TCP and ascii are the default settings which we did not specify explicitly).

- `get __mcrouter__.config_md5_digest`: current config's md5 hash.  Note that this only specifies the main config file's md5 and ignores any additional tracked files.
- `get __mcrouter__.config_sources_info`: md5 hashes for each tracked config file (additional files might be tracked due i.e. `@import` statements).  This is useful for automatic verification that mcrouter is actually running with the correct config version.

  For our example:
  ```
  VALUE __mcrouter__.config_sources_info 0 196
  {
    "mcrouter_config" : "67563d29032fcb3de28a8b0071ac9638",
    "config_file" : "/home/anton/test.json",
    "config_import" : {
      "/home/anton/pools.json" : "c269755d5ddd9a6daac8d64e072c7e9c"
    }
  }
  END
  ```

- `get __mcrouter__.preprocessed_config`: the loaded config with all macros expanded, useful for macro debugging.

  For our example:
  ```
  VALUE __mcrouter__.preprocessed_config 0 288
  {
    "pools" : {
      "bar" : {
        "servers" : [
          "localhost:10002"
        ]
      },
      "foo" : {
        "servers" : [
          "localhost:10001"
        ]
      }
    },
    "route" : {
      "children" : [
        "PoolRoute|foo",
        "PoolRoute|bar"
      ],
      "type" : "AllSyncRoute"
    }
  }
  END
  ```