# SSH

### Generate SSH key pairs

```shell
ssh-keygen -o -a 100 -t ed25519
```

### Default autossh command line arguments

```shell
autossh -M 0 -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3"
```

### Forward remote HTTP ports to the local

```shell
# set GatewayPorts to "yes" or "clientspecified" in /etc/ssh/sshd_config

# -f - Run in background.
# -N - Do not execute a remote command.  This is useful for just forwarding ports.
autossh -f -N -R *:80:127.0.0.1:80 -R *:443:127.0.0.1:443 root@host
```

### Forward local port

```shell
# -f - Run in background.
# -N - Do not execute a remote command.  This is useful for just forwarding ports.
autossh -f -N -L *:22225:zproxy.lum-superproxy.io:22225 root@host
```

### Forward local port to the remote unix socket

```shell
# -f - Run in background.
# -N - Do not execute a remote command.  This is useful for just forwarding ports.
# * - any ip, 127.0.0.1 - listen for the specific ip only.

# docker example
autossh -f -N -L *:3306:/var/run/docker.sock root@host
```

PostgresSQL tunnel:

```shell
autossh -f -N -L *:5432:/var/run/postgresql/.s.PGSQL.5432 root@host

# google cloud
gce ssh a0 -- -N -L *:5432:/var/run/postgresql/.s.PGSQL.5432

psql -h localhost -p 5432
```
