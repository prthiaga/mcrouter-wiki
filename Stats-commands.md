To find out mcrouter state, performance and usage information one can use "stats" command. This information includes: number of requests of each served by mcrouter, replies, how long mcrouter is running, etc. It is a rough analogue of memcached "stats", but with more features.
The most basic usage is to send "stats" request to mcrouter. Assuming mcrouter is running on port 5000:

```bash
 echo stats | nc 0 5000
```

For mcrouter setup from [Home page](Home) possible output is:

```
 STAT version mcrouter 1.0
 STAT commandargs -p 5000 --config-str={"pools":{"A":{"servers":["[::1]:5001"]}},"route":"PoolRoute|A"}
 STAT child_pid 27051
 STAT parent_pid 18758
 STAT time 1410750281
 STAT uptime 29
 STAT num_servers 1
 STAT num_servers_up 0
 STAT num_servers_down 1
 STAT num_clients 1
 ...
```

##stats groups
Mcrouter `stats` command also supports optional `group` parameter. It is passed the same way as `key` for `get`:

```bash
 echo "stats all" | nc 0 5000
```

Group parameter controls which stats will be returned. Valid group names:
* **all**
  Returns all the stats.
* **cmd**
  Returns number of requests received and sent.
* **cmd-in**
  Returns number of requests received.
* **cmd-out**
  Returns number of requests sent.
* **cmd-error**
  Returns number of requests failed.
* **servers**
  This is special group, not included in any of above.
  Returns state of each server from config, together with additional info per each server. Output example:
  ```
   ~# echo stats servers | nc 0 5000
    STAT [0000:0000:0000:0000:0000:0000:0000:0001]:5001:TCP:ascii-1-0 status:up notfound:1 connect_error:2
    END
  ```

To find a list of stats exposed by mcrouter, see [Stats list](Stats-list).