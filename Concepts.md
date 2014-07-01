###Definitions


####Anatomy of a memcache key from mcrouter's point of view

A memcache key is typically a short (< 250 characters) Ascii string which does not contain any whitespace or control characters. Certain characters have a special meaning when parsed by mcrouter.

Let's examine what happens when mcrouter processes the request `GET /region/cluster/foo:key|#|etc`. The prefix `/region/cluster/` is used to lookup the route (see [Configuration](Configuration) for details about route aliases), and is normally stripped before forwarding the request to the destination (there's an option in the pool config that allows preserving this prefix).

The string `|#|` is known as a "hash stop". The suffix up to and including the hash stop is ignored when performing consistent hashing, but it's not stripped. So this request is hashed as `foo:key`, and the destination server will see `GET foo:key|#|etc`. This is useful when you want to group certain keys together on one box.

```
                                      /region/cluster/foo:key|#|etc
Full key:                             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Routing prefix:                       ^^^^^^^^^^^^^^^^
Routing key:                                          ^^^^^^^
Key with stripped routing prefix:                     ^^^^^^^^^^^^^
```

pool is

routing prefix is

hash stop is


###Supported protocols

####ascii

####umbrella

###Supported commands

get

set

delete

explain corresponding reply results:

tko is

data timeout is

connect timeout is

####__mcrouter__

####stats

explanation of what each value means

stat groups: echo "get stats all", echo "get stats detailed", etc.

files written to `/var/mcrouter/stats`