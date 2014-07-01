###AllFastestRoute
Immediately sends the same request to all child route handles. Returns the first non-error reply to come back. Other requests complete in the background and their replies are silently ignored.

Properties:
 * `children`: list of child route handles.


###AllInitialRoute
Immediately sends the same request to all child route handles. Waits for the reply from first route handle in the `children` list and returns it. Other requests are completed asynchronously in the background.

Properties:
 * `children`: list of child route handles.


###AllMajorityRoute
Immediately sends the same request to all child route handles. Waits for replies until a non-error result appears (half + 1) times (or all replies if that never happens), and then returns the latest reply that it sees (the effect is that typically the reply with the most common result is returned).

Properties:
 * `children`: list of child route handles.


###AllSyncRoute
Immediately sends the same request to all child route handles. Collects all replies and responds with the "worst" reply (i.e., the error reply, if any).

Properties:
 * `children`: list of child route handles.


###DevNullRoute
Same as **NullRoute**, but with mcrouter stats reporting.


###ErrorRoute
Immediately returns the error reply for each request. You can specify the error value to return: `"ErrorRoute|MyErrorValueHere"`


###FailoverRoute
Sends the request to the first child in the list and waits for the reply. If the reply is a non-error, returns it immediately. Otherwise, sends the request to the second child, and so on. If all children respond with errors, returns the last error reply.

Properties:
 * `children`: list of child route handles


* **FailoverWithExptimeRoute**
 Failover with additional settings. Sends request to `normal` route handle
 If it responds with an error, checks `settings` and failovers to `failover`
 if necessary.
 Properties:
 * `normal`
 All requests sent here first.
 * `failover`
 List of route handles to which to route in case of error reply from `normal`.
 * `failover_exptime` (int, optional, default 60)
 TTL (in seconds) for update requests sent to failover. Useful when failing
 over "sets" to a special pool with a small TTL to protect against
 temporary outages.
 * `settings` (object, optional)
 This object allows tweaking failover logic, such as specifying which errors from `normal`
 route handle are returned immediately, without sending to `failover` hosts.
 ```JSON
 {
 "tko": {
 "gets": bool (optional, default true)
 "updates": bool (optional, default true)
 "deletes": bool (optional, default false)
 },
 "connectTimeout": { /* same as tko */ },
 "dataTimeout": { /* same as tko */ }
 }
 ```
 "tko", "connectTimeout" and "dataTimeout" correspond to reply errors;
 "gets", "updates", "deletes" correspond to operations. `true` configures 
 mcrouter to failover the request, `false` to return the
 error immediately. For more about errors and operations, see  [here](Routing.md).

* **HashRoute**
 Routes to the destination based on key hash.
 Properties:
 * `children`
 List of child route handles.
 * `salt` (string, optional, default empty)
 Specifies salt to be added to key before hashing
 * `hash_func` (string, optional, default "Ch3")
 Which hashing function to use: "Ch3", "Crc32" or "WeightedCh3"
 **TODO!!!!** explain what each function does.
 * `weights` (list of doubles, valid only when hash_func is 'WeightedCh3')
 Weight for each destination. Must not have fewer elements than the number of
 children. Only the first (number of children) weights are used, though. [??? Clarify last]


* **HostIdRoute**
 Routes to destination based on client host ID.
 Properties:
 * `children`
 List of child route handles.


* **LatestRoute**
 Attempts to "behave well" in how many new targets it connects to.
 Creates a FailoverRoute with at-most `failover_count` child handles chosen
 pseudo-randomly based on client host ID.
 Properties:
 * `children`
 List of child route handles. * `failover_count` (int, optional, default 5)
 Number of route handles to route to.


* **MigrateRoute**
 This route handle changes behavior based on Migration mode.
 1. Before migration starts, sends all requests to `from` route handle.
 2. Between start_time and (start_time + interval), sends all requests except
 deletes to `from` route handle, sends all delete requests to both
 `from` and `to` route handle. For delete requests, returns reply from
 worst of two replies (i.e., any error reply).
 3. Between (start_time + interval) and (start_time + 2*interval), sends all
 requests except deletes to `to` route handle and sends all delete requests
 to both `from` and `to` route handle. For delete requests, returns reply from
 worst among two replies.
 4. After (start_time + 2*interval), sends all requests to `to` route handle.
 Properties:
 * `from`
 Route handle that routes to destination from which we are migrating.
 * `to`
 Route handle that routes to destination to which we are migrating.
 * `start_time` (int)
 Time in seconds from epoch when migration starts.
 * `interval` (int, options, default 3600)
 Duration of migration (in seconds)


* **MissFailoverRoute**
 For get-like requests, sends the same request sequentially to each route
 handle in the list, in order, until the first hit reply.
 If all replies result in errors/misses, returns the reply from the
 last destination in the list.
 **TODO!!!!** What should it do for delete/set? Currently the behavior is strange.
 Properties:
 * `children`
 List of child route handles.
* **NullRoute**
 Returns the default reply for each request right away. Default replies are:
 * delete - not found
 * get - not found
 * set - not stored
 No properties.


* **PoolRoute**
 Route handle that routes to a pool. With different settings, it provides the same
 functionality as HashRoute, but also allows rate limiting, shadowing, et cetera.
 Properties:
 * `pool` (string or object)
 If string, routes to the pool with the specified name. If object, creates a pool on the fly. This object has the same format as 
 the `pools` property described earlier, with an additional `name` property.
 Example:
 ```JSON
 {
 "type": "PoolRoute",
 "pool": {
 "name": "MyPool",
 "servers": [ /* pool hosts here */ ]
 }
 }
 ```
 * `shadows` (optional, default empty)
 List of objects that define additional route handles to which mcrouter should duplicate routed data  (shadow).
 Each object has following properties:
 * `target`
 Route handle for shadow requests.
 * `index_range` (array of two integers, both in [0, number of servers in pool - 1])
 Only requests sent to servers from `index_range` will be sent to `target`,
 * `key_fraction_range` (array of two doubles, both in [0, 1])
 Only requests with key hash from `key_fraction_range` will be sent to
 `target`,
 * `hash` (optional, default "Ch3")
 String or object that defines hash function, same as in **HashRoute**.
 * `rates` (optional, default no rate limiting)
 If set, enables rate limiting requests to prevent server overload.
 The object that defines rate and burst are parameters of a token bucket
 algorithm (http://en.wikipedia.org/wiki/Token_bucket):
 ```JSON
 {
 "type": "PoolRoute",
 "pool": "MyPool",
 "rates": {
 "gets_rate": double (optional)
 "gets_burst": double (optional)
 "sets_rate": double (optional)
 "sets_burst": double (optional)
 "deletes_rate": double (optional)
 "deletes_burst": double (optional)
 }
 }
 ```


* **PrefixPolicyRoute**
Sends to different targets based on specified operations.
 Properties:
 * `default_policy`
 Default route handle.
 * `operation_policies`
 Object, with operation name as key and route handle for specified operation as
 value. Example:
 ```JSON
 {
 "default_policy": "PoolRoute|A",
 "operation_policies": {
 "delete": {
 "type": "AllSyncRoute",
 "children": [ "PoolRoute|A", "PoolRoute|B" ]
 }
 }
 }
 ```
 Sends gets and sets to pool A, sends deletes to pool A and pool B.
 Valid operations are 'get', 'set', 'delete'.


* **WarmUpRoute**
Intended for use as the destination route in a Migrate
 route. Allows for substantial changes to the number of boxes in a pool
 without increasing the miss rate and, consequently, to the load on the
 underlying storage or service.
 All sets and deletes go to the target ("cold") route handle.
 Gets are attempted on the "cold" route handle and, in case of a miss, data is
 fetched from the "warm" route handle (where the request is likely to result
 in a cache hit). If "warm" returns a hit, the response is forwarded to
 the client and an asynchronous request, with the configured expiration time,
 updates the value in the "cold" route handle.
 Properties:
 * `cold`
 Route handle that routes to "cold" destination (with empty cache).
 * `warm`
 Route handle that routes to "warm" destination (with filled cache).
 * `exptime`
 Expiration time for warm up requests.

####Prefix route selector

In addition to selection by routing prefix, mcrouter also allows selection by
key prefix. For example, you might want to route all keys beginning with foo_ to pool A, and all keys starting with bar_ to
pool B. Note that key prefixes are not stripped, unlike routing prefixes.

To enable this logic, the root of the route handle tree should be a special
route handle with the type set to "PrefixSelectorRoute" and the properties as
follows:

* `policies`
 Object with prefix as key and route handle as value. If a key matches
 multiple prefixes, the longest one is selected.
* `wildcard`
 If key prefix doesn't match any prefix in `policies`, requests 
 go here.

Example:
```JSON
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
_Explanation_: requests with routing prefix "/a/a/" and key prefix "a" (but not "ab"!) will be sent to pool A, requests with routing "/a/a/" and key prefix "ab" will be sent to pool B. Other requests with routing prefix "/a/a/" will be sent to pool C. So key "/a/a/abcd" will be sent to pool B (as "abcd"); "/a/a/acdc" to pool A (as "acdc"), "/a/a/b" to pool C (as "b").

####Named handles

You may wish to use the same route handle in different parts of the config. To avoid duplication,  add `name` to route handle object and then refer to this route handle by its name. If two route handles have the same name, only the first is parsed -  the second one is treated as a duplicate and is not parsed (even if it has different properties). I nstead, the first route handle is substituted in its place.

Route handles may also be defined in the `named_handles` property of the config. Example:
```JSON
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
In this example, we specify two pools with different rate limits. Requests with the "/a/a/" routing prefix will be routed to pool A and failover to pool B. Requests  with the "/b/b/" routing prefix will be routed to pool B, and failover to pool A.