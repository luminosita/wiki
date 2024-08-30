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


