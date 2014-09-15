Mcrouter exposes numerous counters which include: number of requests sent, number of replies received, start time, average request duration, etc. These stats are exposed by ["stats" commands](Stats-commands) and in ["stats" file](Stats-files).

Here is an explanation of what most important stats mean:

##Basic stats
* **version**
  Version of mcrouter binary.
* **commandargs**
  Command line used to start mcrouter.
* **time**
  Current server time.
* **child_pid**
  Process id of mcrouter instance.
* **parent_pid**
  Process id of process that started mcrouter.

##stats logged to file
* **asynclog_requests**  
  Number of failed deletes written to spool file. More about spool file read [here](Features#reliable-delete-stream).
* **closed_inactive_connections**  
  Number of connections closed due to inactivity. Once connection is not used for more than a minute,
  mcrouter will force-close it. The period of inactivity is configured with
  [`--reset-inactive-connection-interval`](Command-line-options#--reset-inactive-connection-interval) command line option.
* **cmd_[operation]**  
  Average number of received requests per second drilled down by operation.

* **cmd_[operation]_count**  
  Total number of received requests drilled down by operation.

* **cmd_[operation]_out_failover**  
  Average number of sent failover requests per second drilled down by operation.

* **cmd_[operation]_out_shadow**  
  Number of sent shadow requests per second drilled down by operation.

* **cmd_[operation]_out**  
  Average number of sent normal (non-shadow, non-failover) requests per second drilled down by operation.

* **cmd_[operation]_out_all**  
  Total number of sent requests per second (failover + shadow + normal)

* **cmd_[operation]_out_count**  
  Total number of sent requests per second drilled down by operation.

* **config_age**  
  How long ago (in seconds) mcrouter has reconfigured.

* **config_failures**  
  How many times mcrouter failed to reconfigure (if > 0 and growing, check the config is valid)

* **config_last_attempt**  
  UNIX timestamp of last time mcrouter tried to reconfigure.

* **config_last_success**  
  UNIX timestamp of last time mcrouter reconfigured successfully.

* **dev_null_requests**  
  Number of requests sent to [DevNullRoute](List-of-Route-Handles#devnullroute)

* **duration_us**  
  Average time of processing a request (i.e. receiving request and sending a reply).

* **fibers_allocated**  
  Number of fibers created by mcrouter.

* **proxy_reqs_processing**  
  Proxy requests mcrouter started routing (but didn't receive a reply yet).

* **proxy_reqs_waiting**  
  Requests queued up and not routed yet.

* **result_[reply result]_count**  
  Total number of replies received down by reply result.

* **result_[reply result]_failover**  
  Average number of replies per second received for failover requests drilled down by result.

* **result_[reply result]_shadow**  
  Average number of replies per second received for shadow requests drilled down by result.

* **result_[reply result]**  
  Average number of replies per second received for normal requests drilled down by reply result.

* **result_[reply result]_all**  
  Average number of replies per second received for requests drilled down by reply result.

* **start_time**  
  UNIX timestamp of mcrouter startup

* **uptime**  
  How long ago (in seconds) mcrouter has started.  
