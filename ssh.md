# SSH

## Default autossh command line arguments

```
autossh -M 0 -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3"
```

## Forward remote HTTP ports to the local

```
# set GatewayPorts to "yes" or "clientspecified" in /etc/ssh/sshd_config

# -f - Run in background.
# -N - Do not execute a remote command.  This is useful for just forwarding ports.
autossh -f -N -R *:80:127.0.0.1:80 -R *:443:127.0.0.1:443 root@serp
```

## Forward local port

```
# -f - Run in background.
# -N - Do not execute a remote command.  This is useful for just forwarding ports.
autossh -f -N -L *:22225:zproxy.lum-superproxy.io:22225 root@serp
```

## Forward local port to the remote unix socket

```
# -f - Run in background.
# -N - Do not execute a remote command.  This is useful for just forwarding ports.
# * - any ip, 127.0.0.7 - listen for the specific ip only.
autossh -f -N -L *:3306:/var/run/docker.sock root@serp
```
