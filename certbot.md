# Certbot

### Get certificate using DNS

```shell
apt install -y certbot
```

```shell
certbot certonly --manual --preferred-challenges dns --debug-challenges -d local.softvisio.net
```

Certificates will be placed at:

-   cert: /etc/letsencrypt/archive/local.softvisio.net/fullchain1.pem

-   key: /etc/letsencrypt/archive/local.softvisio.net/privkey1.pem
