### Mounted Folder Accessible in LXC Container

For the mounted folder in the host system 

/mnt/xxx => /dev/sda1

#### NOTE: Important to follow the steps

#### Configure mapping between LXC container and host for user UID/GID 1000:1000 (CT 1000:1000 => host 1000:1000) 
Edit lxc container config file in `/etc/pve/lxc/` folder

Add the following lines:

```
# uid map: from uid 0 map 1005 uids (in the ct) to the range starting 100000 (on the host), so 0..1004 (ct) → 100000..101004 (host)
lxc.idmap = u 0 100000 1000
lxc.idmap = g 0 100000 1000
# we map 1 uid starting from uid 1005 onto 1005, so 1005 → 1005
lxc.idmap = u 1000 1000 1
lxc.idmap = g 1000 1000 1
# we map the rest of 65535 from 1006 upto 101006, so 1006..65535 → 101006..165535
lxc.idmap = u 1001 101001 64530
lxc.idmap = g 1001 101001 64530
```

Edit `/etc/subuid`

```
root:100000:65536
root:1000:1
```

Edit `/etc/subgid`

```
root:100000:65536
root:1000:1
```

#### Make mount accessible by host user UID/GID

```
$ chown -R 1000:1000 /mnt/xxx
```

#### Make bind mount 

```
$ pct set 100 -mp0 /mnt/xxx,mp=/ct-xxx
```

Host mount /mnt/xxx will be accessible through LXC container as /ct-xxx

#### Start LXC container and create user with UID/GID = 1000

```
$ adduser --uid 1000 -gid 1000 <user>
```
