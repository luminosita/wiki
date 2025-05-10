```bash
$ export CA_ROOT_DIR="/home/repo/certs"
$ ca_repo.sh ca -i "/C=SR/ST=Serbia/L=Belgrade/O=Local doo/OU=Local Certificate Authority/CN=Root CA"
Root CA: /home/repo/certs/ca
PEM CA Password: <ca_password>
```

```bash
$ ca_repo.sh intermediate -i "/C=SR/ST=Serbia/L=Belgrade/O=Local doo/OU=Local Certificate Authority/CN=Local Intermediate CA" -n int
Root CA: /home/repo/certs/ca, Intermediate CA: /home/repo/certs/ca/int
PEM CA Password: <ca_password>
```

```bash
$ ca_repo.sh server -i "/C=SR/ST=Serbia/L=Belgrade/O=Local doo/OU=Local Vault Service/CN=vault.lan" -n int -p "IP:172.16.20.11,IP:172.16.20.12,IP:172.16.20.13"
Intermediate CA: /home/repo/certs/ca/int
PEM CA Password: <ca_password>
```