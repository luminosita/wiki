
# Trusted CA SSH Certificates

### Generate CA Key

```bash
$ ssh-keygen -f ~/.ssh/ca_user_key
```

### Setup SSHD Server

Copy the public key into an appropriate location.

```bash
$ scp ~/.ssh/ca_user_key.pub <user>@<ssh server>:/etc/ssh/
```

Update the sshd_config to add the TrustedUserCAKeys option and restart the service.

```bash
$ vim /etc/ssh/sshd_config
...
TrustedUserCAKeys /etc/ssh/ca_user_key.pub
...

$ systemctl restart sshd
```

### Sign Client Public SSH Key

Create client SSH key-pair or use the existing

```bash
$ ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_test_rsa
```

The following command with sign existing SSH public key and create certificate `~/.ssh/id_test_rsa-cert.pub`
Certificate will be valid for one hour (-V +1h), with the serial number `1`, server user account `ubuntu` and 
certificate identity `test` Certificate identity will show up in server SSH log file for the purpose of audit 

```bash
$ ssh-keygen -s ~/.ssh/ca_user_key -I test -n ubuntu -V +1h -z 1 ~/.ssh/id_test_rsa.pub
```

### Login

```bash
$ ssh -i ~/.ssh/id_test_rsa ubuntu@<server ip>
```

### Multiple Account Access

Certificate can enable access to multiple accounts on one SSH server
There is a `dba` user account and we will use a principal named `zone-databases` to allow access to the `dba` user account. Any certificate which is created with the `zone-databases` principal will allow the relevant user access to the `dba` user.
Firstly add the AuthorizedPrincipalsFile to the sshd_config file and restart the service.

```bash
$ vim /etc/ssh/sshd_config
...
AuthorizedPrincipalsFile /etc/ssh/auth_principals/%u

$ systemctl restart sshd
```

The `auth_principals` directory also needs to be created. Within here the `dba` user file will be created with the zone-databases principal present.

```bash
$ mkdir /etc/ssh/auth_principals/
$ echo "zone-databases" > /etc/ssh/auth_principals/dba
```

The certificate will now be created with the zone-databases principal added.

```bash
$ ssh-keygen -s ~/.ssh/ca_user_key -I test -n ubuntu,zone-databases -V +1h -z 1 ~/.ssh/id_test_rsa.pub
```

Test login

```bash
$ ssh -i ~/.ssh/id_test_rsa dba@<server ip>
```