A flavor refers to a specific mcrouter configuration. Flavor configs specify command line arguments for running mcrouter. They configure which [config file](Config-File) to use, and various other startup parameters. For a full list of mcrouter startup options see [Startup Options](Command-line-options).

In practice, flavors are JSON files that contains all (or some) of the command-line options one would use to run mcrouter. They are commonly used to avoid long command-lines and/or to save configs. In flavors, we use _internal name_ instead of _command line option_. In most cases it is the same as long option name with dashes replaced with underscores (i.e. --disable-reload-configs becomes disable_reload_configs).

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
There are some things from the above example that are worth mentioning:  
* Althought the file is named `myflavor-standalone`, the flavor named informed is simply `myflavor`. That's a rule for standalone flavors.  
* The "libmcrouter_options" object is optional for standalone flavor configs. If present, these options will be used to replace options read from the libmcrouter flavor config for that particular flavor.  
* The following options are _standalone options_ and should be informed in the _standalone_options_ object of the standalone flavor file: `log_file, debug_level, ports, ssl_ports, listen_sock_fd, pidfile, background, managed, fdlimit, max_global_outstanding_reqs, max_client_outstanding_reqs, requests_per_read, enable_failure_logging`. All the other options should go into the libmcrouter_options object or in the libmcrouter flavor file.  

# Usage
To use a flavor, simply inform it's path (without any suffixes, i.e. "-standalone") to mcrouter command line:
```bash
$ mcrouter /path/to/flavor
```

# Behavior
If just one of the flavor files are present (either the standalone or the libmcrouter file), the behavior is pretty straight forward: the options present in that file are loaded and applied. (note that the suffix "-standalone" **must** be present to the _standalone flavor file name_ but should **never** be specified in the command-line).  
If the user has both files (the libmcrouter **and** the standalone flavor files) mcrouter first loads the options present in the **libmcrouter flavor file**. Then, mcrouter locates the **standalone flavor file** by appending the suffix "-standalone" to the end of the informed flavor file name. It is then loaded and, any options found in the "libmcrouter_options" object overwrites options with the same name read from "options" object at the **libmcrouter flavor file**.

# Example
Find bellow a simply scenario that illustrates how mcrouter handles flavors:

File /var/mcrouter/myflavor:
```javascript
{
  "options": {
    "default_route": "/region1/cluster1/",
    "config_file": "mcrouter_config.json"
  }
}
```

File /var/mcrouter/myflavor-standalone:
```javascript
{
  "libmcrouter_options": {
    "default_route": "/region3/"
  },
  "standalone_options": {
    "ports": "5000"
  }
}
```


Given that the above files are properly saved to disk at the locations specified, the following two commands are equivalent:
```bash
$ mcrouter /var/mcrouter/myflavor

$ mcrouter --default-route='/region3/' --config-file='mcrouter_config.json' -p 8888
```

**Note**: The _default_route_ option present in the _libmcrouter_options_ object in the **standalone flavor file**, overwrote the _default_route_ option present in the **libmcrouter flavor file**.
