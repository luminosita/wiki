## Step 1: prepare the host

Because LXC containers share the host’s kernel, we have to prepare the host. This means disabling the swap and also loading a couple of modules.

### Sysctl changes

First I adapt the sysctl file on the host:

```
$ vim /etc/sysctl.conf
```

Uncomment the following line:

```
#net.ipv4.ip_forward=1
```

And add:

```
vm.swapiness=0
```

### Disable swap:

swapoff -a
After that adapt the fstab file:

vim /etc/fstab

> **_NOTE:_** Disable swap on host
I did a reboot of the full host, I don’t know if this is needed but wanted to be sure…

> **_ALTERNATIVE SOlUTION:_**
```
$ systemctl restart systemd-sysctl
$ systemctl daemon-reload
```