Mcrouter will interpret certain parts of the client request's key in a special way.

A memcached key is typically a short (mcrouter limit is 250 characters) ASCII string which does not contain any whitespace or control characters.

Consider the request `get /region/cluster/foo:key|#|etc`. The prefix `/region/cluster/` is [[used to lookup the route|Routing prefix]], and is normally stripped before forwarding the request to the destination (there's a `keep_routing_prefix` option in the pool config that allows preserving this prefix, which is useful when setting up multiple mcrouters in series).

### Hash stop
The string `|#|` is known as a "hash stop". The suffix up to and including the hash stop is ignored when performing consistent hashing, but it's not stripped. So this request is hashed as `foo:key`, and the destination server will see `get foo:key|#|etc`. This is useful when you want to group certain keys together on one box.

### Admin requests
A key that starts with a special prefix `__mcrouter__.` is interpreted as an [[admin request|Admin requests]] and is not sent to any destination. For example, `get __mcrouter__.version` will cause mcrouter to return its package string (like `mcrouter 1.0`) as value.

### Key syntax summary
```
                                      /region/cluster/foo:key|#|etc
Full key:                             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Routing prefix:                       ^^^^^^^^^^^^^^^^
Routing key:                                          ^^^^^^^
Key with stripped routing prefix:                     ^^^^^^^^^^^^^
```
