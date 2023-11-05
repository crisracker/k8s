## Add user to Ubuntu

```cmd

adduser xxx

```

## Assign password to existing user to Ubuntu

```cmd

passwd <username>

```

## Update the sshd config to allow password authentication

```cmd

vi /etc/ssh/sshd_config

```

```
PasswordAuthentication --> change to Yes

restart service -- > sudo systemctl restart sshd
```

## Copy files from node to node

```cmd

sudo scp admin.conf chris@k8s-worker1:/home/chris

```

