Proxmox API tokens:
Priviledge Separation - unchecked

VM Template Config
```
agent: 1
bios: ovmf
boot: order=virtio0;ide2
cicustom: user=local:snippets/user.yaml,vendor=local:snippets/vendor.yaml
cipassword: **********
ciupgrade: 1
ciuser: ubuntu
cores: 1
cpu: host
efidisk0: local-lvm:base-8010-disk-0,pre-enrolled-keys=0,size=4M
ide2: local-lvm:vm-8010-cloudinit,media=cdrom
ipconfig0: ip=109.245.66.169/24,gw=192.168.50.1
ipconfig1: ip=dhcp
machine: q35
memory: 1024
meta: creation-qemu=9.0.2,ctime=1724877316
name: ubuntu-2404-cloudinit-template
net0: virtio=BC:24:11:A5:56:50,bridge=vmbr0
net1: virtio=BC:24:11:41:C6:A6,bridge=vmbr0
ostype: l26
scsihw: virtio-scsi-pci
serial0: socket
smbios1: uuid=57a4901a-4a06-4174-b058-f32701df9474
sockets: 1
sshkeys: %0Assh-rsa%20AAAAB3NzaC1yc2EAAAADAQABAAACAQDBz1PKlnjPYHmkqrct9P0A6moYjOcO7%2FAfsrbgf8XeJ3e3AMmxGzmfFEoTHy5kUQgZDvXGFWcbWI6PmcRbtcTAmEQkfdVrREVoH67tAQsIZNdyDB3QLxg2gWrHb%2Fn1AzgFAC8HKjH9AgT%2B%2FLshLqmYD6QoZnNpI68ssjeRnyvtfuVlPsowHrcweHt%2FK%2Fj5BnVtN4kpFniwDw3BbycRj0DMLh%2BvMiEMkI%2BHNV6XBDW9UL8DS3Bmfmd2D1u3o%2FTVtYZ9qQT2PllE74qI1ojMAdm6tky491Buj43xq2ot0kuV08CNeB0Vjmxlm8vZqEvTgH7NcoIUdnhp3mAcbVkEivc0vUpZGrBn3YFEiTL56P%2BBnUx9bbYZWXRrU3tECRw2ALOVvXyTBQ0GT49cNzG%2Blt3m0JWnAbWUnHgJmKm0B1DKTkM0o1aabLiok%2FrkHaTvQjXiD9sQeiXzIROodROBgSwiKYqJFHEXet2HsDuVz%2FdalzSgRFi8w89lDgGe2ByPmWUHQ1J%2B9WGviXonXz96O6%2FLsooJDONaFkPRzZ2wSon85RB78QAlVTNs%2FsiU6DFLLPm0dZoeDy92y66TTDCsjt%2FuiAk3f5gUTwgaED3vsux%2FVLRcOnbCehu5eZ50ePek7%2BJuJQs39OYWMZNy1vOdpnsXu1uUs%2FFO%2Bn6KwgZdHMppxw%3D%3D%20root%40proxmox%0A
tags: 24.04;k3s;ubuntu;ubuntu-noble
template: 1
vga: serial0
virtio0: local-lvm:base-8010-disk-1,discard=on,size=32G
vmgenid: 870ed80e-4fd8-4573-a1f0-4cdf02e1e48c
```

root@proxmox:/var/lib/vz/snippets#

cat user.yaml 
```
#cloud-config
users:
  - name: k3s
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    passwd: ///ux8kmQAxPJI.xvAiffrewbaytayDzMFh.TMP.NSGU4UXLYoLGE9Bx52D.
    lock-passwd: false
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDFbfs3Ga7TDc5UrcGR4Ujs/s6vbWGXzNeizYaYe3qkANHrY3HM3D2BFKwH+VGukXdYtPT5jkBxX38/739NYCYTtXoD9OsA8bD4DoHajtcJrbp6ZYMwnFvnpIrT3GvmM8KWFKbOWAVQY2rc1tIuObLZciS+pUfSloEf4KHbWth/rBA/sPZEwBJHjHacMGdevNb1RdKIwu6K1qHu9jvtsZswxNalINQb9unYnkJ7yoOrV/weASv/NEDEGJreJ7o1nS9VRAVPgJd0pcWOxRHbvAOwkW20oQ2mU0LMHw+RcSS+IY22g4fXrgxMXx+eEC6wSY7Y6v9Pw0e+qeFDNVJ2hMfo2f2iMgK+21Je8/gXAxTNdEpu64DWKQeplLEyb4ksEBLq/CrcH3bfZNoxOris/g+meXm/J+U11sFbBAFFwh4prf7o4PZZFqhjxck4p95pl+kpL5oy/qiG4qUteUTgizQKZkjK85YWhcLTYsDltjVxYArKrUYuEZQA7awgn6aXnauhzXeT+aCCL6yBzuYSsLK+KYGwdOtBu0hdn0i1d4aCkI+2KlAdEPMrtDsbv8n9JWWQmQkQVSwaTWvVQpMeSRfqOjZLRXtkQHcjCR3tUbE4XG4d7dyDD1iCsEA2i8BQXzxdGGyu4NsDo368k8m7h9NT1vJkEOuTtsXYDNkRXm/54Q== milosh@Gianni
```
cat vendor.yaml
```
#cloud-config  
runcmd:  
    - apt update  
    - apt install -y qemu-guest-agent  
    - systemctl start qemu-guest-agent  
    - reboot  
```

