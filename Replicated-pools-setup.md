In many cases in which cache is deployed at large scales, frequently-accessed data are read by huge numbers of clients, and memcached servers risk running out of connections. Problems may also arise when critical data must always be available, even when several servers go down. Mcrouter solves both of these problems with [replication](http://en.wikipedia.org/wiki/Replication_(computing)).

Note: since you decided to use cache, we assume that the number of gets in your deployment is much higher than number of sets and deletes.

Lets reword the problem. Using more specific terms, we want to:
* send gets to a random box from a [pool](Pools). If request fails, get the data from any other box.
* send both sets and deletes to all hosts inside a [pool](Pools).

Here is a sample mcrouter configuration to achieve this:

```JavaScript
 {
   "pools": {
      "A": {
         "servers": [
            // hosts of replicated pool, e.g.:
           "127.0.0.1:12345",
           "[::1]:12346"
         ]
      }
   },
   "route": {
     "type": "OperationSelectorRoute",
     "operation_policies": {
       "delete": "AllSyncRoute|Pool|A",
       "add": "AllSyncRoute|Pool|A",
       "get": "LatestRoute|Pool|A",
       "set": "AllSyncRoute|Pool|A"
     }
   }
 }
```

_Explanation_: deletes and sets are sent to all hosts in pool A, gets are sent to a random host in the pool based on mcrouter's host id (thus, to split the load, you'll need to run several mcrouter instances). If a get request fails, it is automatically sent (retried) to another host in pool, then another, and so on. By default, number of retries is 5.

Find out more about mcrouter configuration [here](Configuration).