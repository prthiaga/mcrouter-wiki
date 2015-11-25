Mcpiper is a debugging tool for mcrouter. It allows analysis of all the traffic that goes in and out mcrouter. 

Differently from other similar tools, mcpiper is not a network sniffer (and do not require `sudo` to run). Instead, mcrouter creates a series debug fifos (aka named pipes), that mcpiper connects to read the data. Mcrouter automatically starts replicating network traffic to those fifos as soon as mcpiper starts to run.

## Setup
If you haven't done it yet, you will need to follow the [mcrouter installation instructions](https://github.com/facebook/mcrouter/wiki/mcrouter-installation). Those instructions will build and install both mcrouter and mcpiper.

By default, mcrouter will try to create the debug fifos in `/var/mcrouter/fifos/` directory. To change that directory, use the `--debug-fifo-root` argument. Make sure that the directory being used exists and is writable.

Now, simply start mcpiper and you should see data flowing. 
```bash
$ mcpiper 
# If you started mcrouter using another directory as the debug fifos root:
$ mcpiper --fifo-root="<PATH>"
```

## Common use cases
Search for an expression. Will match everything that **contains** the given BRE (basic regular expression):
```bash
$ mcpiper <BRE>
# Examples:
$ mcpiper "test" # Search for the string "test"
$ mcpiper a.c # Serach for any string that match the "a.c" BRE.
$ mcpiper -i abc # Do a case insensitive search for "abc"
```

Filter IP Address / Port:
```bash
$ mcpiper -H=<HOST> -p=<PORT>
# Example:
$ mcpiper -H "127.0.0.1" -p 5000
```

Only investigate fifos which the file name match the given pattern (BRE):
```bash
$ mcpiper --filename-pattern=<BRE> # or simply -f
# Examples:
$ mcpiper --filename-pattern="client" # Investigate client fifos (i.e. data sent to memcached).
$ mcpiper --filename-pattern="server" # Investigate server fifos (i.e. data received by mcrouter)
```

Do not print the values:
```bash
$ mcpiper -q
```


For more information on how to use mcpiper:
```bash
$ mcpiper --help
```