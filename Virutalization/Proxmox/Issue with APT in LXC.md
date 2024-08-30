# APT issue with mapped host/container user/group

```
$ apt update
E: setgroups 65534 failed - setgroups (22: Invalid argument)
E: setegid 65534 failed - setegid (22: Invalid argument)
Reading package lists... Done
E: setgroups 65534 failed - setgroups (22: Invalid argument)
E: setegid 65534 failed - setegid (22: Invalid argument)
E: Method gave invalid 400 URI Failure message: Failed to setgroups - setgroups (22: Invalid argument)
E: Method gave invalid 400 URI Failure message: Failed to setgroups - setgroups (22: Invalid argument)
E: Method http has died unexpectedly!
E: Sub-process http returned an error code (112)
```

### FIX:

Add the following line into LXC container config file in `/etc/pve/lxc/xxx.conf`

lxc.idmap: g 65534 165534 1