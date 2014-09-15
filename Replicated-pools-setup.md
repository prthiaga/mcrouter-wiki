When using cache at a large scale it happens that most frequently used data are read by huge amount of clients and memcached servers simply run out of connections. Another problem arises when some data is so important that should be available always, even when several servers go down. Since you decided to use cache, we assume that number of gets is much higher than number of sets and deletes.

Both problems are solved with [replication](http://en.wikipedia.org/wiki/Replication_(computing)).

Lets reword the problem. Using more specific terms, we want to:
* send gets to a random box from a [pool](Pools). If request fails, get the data from any other box.
* send both sets and deletes to all hosts inside a [pool](Pools).

Here is a sample mcrouter configuration to achieve this:

```JavaScript
 {
   "pools": {
     // hosts of replicated pool, e.g.:
     "127.0.0.1:12345",
     "[::1]:12346"
   },
   "route": {
     "type": "PrefixPolicyRoute",
     "operation_policies": {
       "delete": "AllSyncRoute|Pool|A",
       "get": "LatestRoute|Pool|A",
       "set": "AllSyncRoute|Pool|A"
     }
   }
 }
```

_Explanation_: deletes and sets are sent to all hosts in pool A, gets are sent to a random host in the pool based on mcrouter's host id (thus, to split the load, you'll need to run several mcrouter instances). If a get request fails, it is automatically sent (retried) to another host in pool, then another, and so on. By default, number of retries is 5.

Find out more about mcrouter configuration [here](Configuration).