A memcache key is typically a short (mcrouter limit is 250 characters) ASCII string which does not contain any whitespace or control characters. Certain characters have a special meaning when parsed by mcrouter.

Consider the request `GET /region/cluster/foo:key|#|etc`. The prefix `/region/cluster/` is [[used to lookup the route|Routing prefix]], and is normally stripped before forwarding the request to the destination (there's a `keep_routing_prefix` option in the pool config that allows preserving this prefix, which is useful when setting up multiple mcrouters in series).

The string `|#|` is known as a "hash stop". The suffix up to and including the hash stop is ignored when performing consistent hashing, but it's not stripped. So this request is hashed as `foo:key`, and the destination server will see `GET foo:key|#|etc`. This is useful when you want to group certain keys together on one box.

### Key syntax summary
```
                                      /region/cluster/foo:key|#|etc
Full key:                             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Routing prefix:                       ^^^^^^^^^^^^^^^^
Routing key:                                          ^^^^^^^
Key with stripped routing prefix:                     ^^^^^^^^^^^^^
```
