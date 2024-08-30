### Disable Swap

First I adapt the sysctl file on the host:

vim /etc/sysctl.conf
Uncomment the following line:

And add:

vm.swapiness=0
Disable swap:

swapoff -a
After that adapt the fstab file:

vim /etc/fstab

> **_NOTE:_** Disable swap on host
I did a reboot of the full host, I don’t know if this is needed but wanted to be sure…

> **_ALTERNATIVE SOlUTION:_**
```

```