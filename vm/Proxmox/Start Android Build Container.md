### Preconfiguration

#### Mount Android source code disk/partition

```
$ mount /dev/sdb1 /mnt/android
```

#### Start LXC container

From Proxmox UI start container for Android build

Login with `root` and switch to `build` user

#### Start Docker container for VS Code

```
$ docker compose up -d
```

#### Access VS Code via `http://proxmox.wan:8080` and password `laptop01`

#### Enable Swap

Edit `/etc/sysctl.conf`, comment the following line at the bottom of the file:

vm.swapiness=0

Enable swap:

swapon -a

After that uncomment swap line in the fstab file `/etc/fstab`

```
#/dev/pve/swap none swap sw 0 0
```

Reload services ...

```
$ systemctl restart systemd-sysctl
$ systemctl daemon-reload
```



