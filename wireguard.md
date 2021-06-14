# Server

### Install on CentOS

```shell
dnf install -y elrepo-release epel-release
dnf install -y kmod-wireguard wireguard-tools iptables
```

### Setup

```shell
NETWORK_ADDR=10.0.0.1/24
POSTROUTING_ETH=eth0

# generate server keys
umask 077
mkdir -p /etc/wireguard
pushd /etc/wireguard

# generate server keys
wg genkey | tee /etc/wireguard/server.private.key | wg pubkey >/etc/wireguard/server.public.key

# generate server config
cat <<EOF >/etc/wireguard/wg0.conf
[Interface]
PrivateKey = $(cat /etc/wireguard/server.private.key)
Address    = $NETWORK_ADDR
ListenPort = 51820
PostUp     = sysctl -w net.ipv4.ip_forward=1; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o $POSTROUTING_ETH -j MASQUERADE
PostDown   = iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o $POSTROUTING_ETH -j MASQUERADE
# SaveConfig = true
EOF

# cleanup
rm -rf /etc/wireguard/server.private.key

# enable and run service
systemctl enable wg-quick@wg0.service
systemctl restart wg-quick@wg0.service

# configure interface
# wg set wg0 private-key /etc/wireguard/server.private.key listen-port 51820
# wg-quick save wg0
```

### Client

```shell
ENDPOINT=dev.creationshop:51820
CLIENT_ADDR=10.0.0.10

# generate client keys
wg genkey | tee /etc/wireguard/client.private.key | wg pubkey >/etc/wireguard/client.public.key
wg genpsk >/etc/wireguard/client.preshared.key

# add peer
wg set wg0 peer $(cat /etc/wireguard/client.public.key) persistent-keepalive 60 allowed-ips $CLIENT_ADDR/32

# add peer with pre-shared key. !!! currently not works for windows client !!!
# wg set wg0 peer `cat /etc/wireguard/client.public.key` preshared-key /etc/wireguard/client.preshared.key persistent-keepalive 60 allowed-ips 10.0.0.0/24

wg-quick save wg0

# generate client config
cat <<EOF >/etc/wireguard/client.wireguard.conf
[Interface]
PrivateKey          = $(cat /etc/wireguard/client.private.key)
Address             = $CLIENT_ADDR/32
DNS                 = 1.1.1.1
MTU                 = 1420

[Peer]
PublicKey           = $(cat /etc/wireguard/server.public.key)
Endpoint            = $ENDPOINT
AllowedIPs          = 0.0.0.0/0, ::/0
PersistentKeepalive = 60
# PresharedKey        = $(cat /etc/wireguard/client.public.key)
EOF

rm -rf /etc/wireguard/client.private.key /etc/wireguard/client.public.key /etc/wireguard/client.preshared.key
```

### AllowedIPs

On client if you want to tunnel all traffic - use `0.0.0.0/0`, or if you want tot route only traffic to specific network via wireguard - use this network address, eg: `10.10.10.0/24`.

### Docker

```shell
d run -it --cap-add=NET_ADMIN --cap-add=SYS_ADMIN -p 51820:51820/tcp -p 51820:51820/udp --network private softvisio/core

dnf install -y elrepo-release epel-release
dnf install -y kmod-wireguard wireguard-tools iptables

# remove "sysctl -w net.ipv4.ip_forward=1" from PostUp

# wireguard kernel module must be installed on the host machine.
```
