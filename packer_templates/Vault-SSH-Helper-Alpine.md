# Vault SSH Helper (Alpine)

### Install packages

```bash
$ apk add make gcc libc-dev git
```

### Install Go

```bash
$ wget https://go.dev/dl/go1.24.2.linux-amd64.tar.gz
$ tar -C /usr/local -xzf go1.24.2.linux-amd64.tar.gz
$ export PATH=$PATH:/usr/local/bin/go:~/go/bin
```

### Build Vault SSH Helper

```bash
$ git clone https://github.com/hashicorp/vault-ssh-helper.git
$ cd vault-ssh-helper
$ go install github.com/mitchellh/gox
$ make
```

Library is located in `bin` folder

