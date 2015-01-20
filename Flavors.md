A flavor refers to a specific mcrouter configuration. Flavor configs specify command line arguments for running mcrouter. They configure which [config file](Config-File) to use, and various other startup parameters. For a full list of mcrouter startup options see [Startup Options](Command-line-options).

In practice, flavors are JSON files that contains all (or some) of the command-line options one would use to run mcrouter. They are commonly used to avoid long command-lines and/or to save configs. In flavors, we use _internal name_ instead of command line option. In most cases it is the same as long option name with dashes replaced with underscores (i.e. --disable-reload-configs becomes disable_reload_configs).

There are two kinds of flavors: libmcrouter flavors (or simply flavors) and standalone mcrouter flavors, which are described in details bellow.

## libmcrouter flavor
The libmcrouter flavor (or simply flavor) config specifies common mcrouter options. See the example bellow:
```javascript
{
  "options": {
    "default_route": "/region1/cluster1/",
    "config_file": "mcrouter_config.json"
  }
}
```
Assuming the above config is saved in a file named `myflavor`, the command `mcrouter myflavor -p 5000` is equivalent to `mcrouter --default-route='/region1/cluster1/' --config-file='mcrouter_config.json' -p 5000`.

## standalone flavor
The standalone flavor config can be used to specify parameters related to the _mcrouter process_. See the example bellow:
```javascript
{
  "libmcrouter_options": {
    "default_route": "/region1/cluster1/",
    "config_file": "mcrouter_config.json"
  },
  "standalone_options": {
    "log_file": "/var/logs/mcrouter-web.log",
    "ports": "8888"
  }
}
```
Assuming the above config is saved in a file named `myflavor-standalone`, the command `mcrouter myflavor` is equivalent to `mcrouter --default-route='/region1/cluster1/' --config-file='mcrouter_config.json' --log-file='/var/logs/mcrouter-web.log' -p 8888`. 
There are some things that are important to be mentioned from the above example:  
* Althought the file is named `myflavor-standalone`, the flavor named informed is simply `myflavor`. That's a rule for standalone flavors.  
* The "libmcrouter_options" object is optional for standalone flavor configs. If not present, these options will be retrieved from the libmcrouter flavor config for that particular flavor.  
* The following options are _standalone options_ and should be informed in the _standalone_options_ object of the standalone flavor file: `log_file, debug_level, ports, ssl_ports, listen_sock_fd, pidfile, background, managed, fdlimit, max_global_outstanding_reqs, max_client_outstanding_reqs, requests_per_read, enable_failure_logging`. All the other options should go into the libmcrouter_options object or in the libmcrouter flavor file.  
