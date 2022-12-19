# Google cloud

Links:

-   [Cloud DNS](https://cloud.google.com/compute/docs/internal-dns)
-   [Cloud IP addresses](https://cloud.google.com/compute/docs/ip-addresses)
-   [Cloud init](https://www.digitalocean.com/community/tutorials/an-introduction-to-cloud-config-scripting)

NOTE:

-   `gc` is the alias for `gcloud`;
-   `gce` is the alias for `gcloud compute`;
-   `gcp` is the alias for `gcloud config configurations`;

## Init gcloud

```shell
gc init --console-only
```

## Create project

Project id must be unique across all google cloud platform. So use project id in `<organization-name>-<project-name>` format.

Create project:

```shell
gc projects create <project-id>
```

Link billing account:

```shell
gc beta billing accounts list

gc beta billing projects link <project-id> --billing-account <billing-account-id>
```

After project is created you need to create project configuration file at `~/.config/gcloud/configurations/config_<project_name>`.

Set project region and zone according to the ping latency value [https://www.gcping.com](https://www.gcping.com).

Activate project configuration:

```shell
gcp activate <project-name>
```

## Change project billing

1. Goto [https://console.cloud.google.com/cloud-resource-manager](https://console.cloud.google.com/cloud-resource-manager).

2. In left-side panel add new user with the following roles: `Browser`, `Project Billing Manager`.

3. New user must to change billing account for this project to the own account here: [https://console.cloud.google.com/billing/projects](https://console.cloud.google.com/billing/projects).

## Initialize cluster

```shell
# create default ssl policies
gce ssl-policies create compatible --profile=COMPATIBLE
gce ssl-policies create modern --profile=MODERN
gce ssl-policies create restricted --profile=RESTRICTED

# create default healthchecks
gce health-checks create tcp tcp --use-serving-port
gce health-checks create http http --use-serving-port --request-path=/healthcheck

# create default url maps
gce url-maps import http --global --source=http.url-map.yaml

# disable default firewall rules
gce firewall-rules update default-allow-rdp --disabled

# allow traffic from Google cloud load balancers and health checkers
gce firewall-rules create allow-gcloud-load-balancer --source-ranges=35.191.0.0/16,130.211.0.0/22 --action=ALLOW --rules=tcp,udp

# allow http traffic from CloudFlare ipv4
yes | gce firewall-rules delete allow-cloudflare-ipv4-http
gce firewall-rules create allow-cloudflare-ipv4-http --source-ranges=$(curl -fsSL https://www.cloudflare.com/ips-v4 | xargs | sed -e "s/ /,/g") --action=ALLOW --rules=tcp:80

# allow http traffic from CloudFlare ipv6
yes | gce firewall-rules delete allow-cloudflare-ipv6-http
gce firewall-rules create allow-cloudflare-ipv6-http --source-ranges=$(curl -fsSL https://www.cloudflare.com/ips-v6 | xargs | sed -e "s/ /,/g") --action=ALLOW --rules=tcp:80

# reserve ip addresses for load balancer
gce addresses create public-ipv4 --ip-version=IPV4 --global
gce addresses create private-ipv4 --ip-version=IPV4 --global

# create nat
gce routers create nat --network=default
gce routers nats create nat --router=nat --auto-allocate-nat-external-ips --nat-all-subnet-ip-ranges --enable-dynamic-port-allocation
```

## Create instances

```shell
gce instances create a0 \
    --machine-type=e2-micro \
    --image-project=ubuntu-os-cloud --image-family=ubuntu-minimal-2204-lts \
    --network-interface=no-address \
    --metadata-from-file user-data=cloud-init.yaml
```

To create instance with external reserved ip address assigned replace:

```shell
--network-interface=no-address
```

with:

```shell
--address=public-ipv4
```

where `public-ipv4` must be regional (created without `--global` flag).

## Create certificates

Before start:

-   Create `dns` records for all used domains, using load valancer ip addresses.
-   On `cloudflare` for each domain separately add CAA records for "pki.goog".

Create certificate:

<!-- prettier-ignore -->
```shell
gce ssl-certificates create <domain-com> --global --domains= <domain.com>
```

Check certificates status:

```shell
gce ssl-certificates list
```

## Create instances group

```shell
gce instance-groups unmanaged create nginx
gce instance-groups unmanaged add-instances nginx --instances=a0
gce instance-groups set-named-ports nginx --named-ports=http:80,pgsql:5432,proxy:8085
```

## HTTP load balancer

Create redirect from `http` to `https`:

```shell
gce target-http-proxies create http --url-map=http --global
gce forwarding-rules create http --load-balancing-scheme=EXTERNAL --address=public-ipv4 --ports=80 --target-http-proxy=http --global
```

Remove:

```shell
yes | gce forwarding-rules delete http --global
yes | gce target-http-proxies delete http
yes | gce url-maps delete http
```

## HTTPS load balancer

Create backend service:

```shell
gce backend-services create nginx \
    --global \
    --global-health-checks --health-checks=tcp \
    --protocol=HTTP --port-name=http --timeout=60s \
    --custom-response-header="Strict-Transport-Security:max-age=63072000; includeSubdomains; preload" \
    --cache-mode=USE_ORIGIN_HEADERS --enable-cdn --serve-while-stale=0

gce backend-services add-backend nginx --global --instance-group=nginx
```

Create load balancer:

<!-- prettier-ignore -->
```shell
gce url-maps create https --default-service=nginx
gce target-https-proxies create https --url-map=https --ssl-policy=modern --ssl-certificates=<domain-com>
gce forwarding-rules create https --load-balancing-scheme=EXTERNAL --address=public-ipv4 --ports=443 --target-https-proxy=https --global
```

Remove:

```shell
yes | gce forwarding-rules delete https --global
yes | gce target-https-proxies delete https
yes | gce url-maps delete https

yes | gce backend-services delete nginx --global
```

## PostgreSQL service

Create backend service:

```shell
gce backend-services create pgsql --global --global-health-checks --health-checks=tcp --protocol=TCP --port-name=pgsql --timeout=600s
gce backend-services add-backend pgsql --global --instance-group=nginx
```

Create SSL load balancer (not works with `psql` client, need to create SSL tunnel first):

<!-- prettier-ignore -->
```shell
gce target-ssl-proxies create pgsql --backend-service=pgsql --ssl-policy=restricted --ssl-certificates=<domain-com>
gce forwarding-rules create pgsql --load-balancing-scheme=EXTERNAL --address=private-ipv4 --ports=5432 --target-ssl-proxy=pgsql --global
```

Remove:

```shell
yes | gce forwarding-rules delete pgsql --global
yes | gce target-ssl-proxies delete pgsql

yes | gce backend-services delete pgsql --global
```

## Proxy service

Create backend service:

```shell
gce backend-services create proxy \
    --global \
    --global-health-checks --health-checks=tcp \
    --protocol=TCP --port-name=proxy --timeout=60s

gce backend-services add-backend proxy --global --instance-group=nginx
```

Create TCP:8085 proxy

```shell
gce target-tcp-proxies create proxy-tcp --proxy-header=PROXY_V1 --backend-service=proxy
gce forwarding-rules create proxy-tcp --load-balancing-scheme=EXTERNAL --address=public-ipv4 --ports=8085 --target-tcp-proxy=proxy-tcp --global
```

Create SSL:8099 proxy:

```shell
gce target-ssl-proxies create proxy-ssl --backend-service=proxy --proxy-header=PROXY_V1 --ssl-policy=restricted --ssl-certificates=proxy-softvisio-net
gce forwarding-rules create proxy-ssl --load-balancing-scheme=EXTERNAL --address=public-ipv4 --ports=8099 --target-ssl-proxy=proxy-ssl --global
```

Remove:

```shell
yes | gce forwarding-rules delete proxy-tcp --global
yes | gce target-tcp-proxies delete proxy-tcp

yes | gce forwarding-rules delete proxy-ssl --global
yes | gce target-ssl-proxies delete proxy-ssl

yes | gce backend-services delete proxy --global
```

## Machine type

Default type is: `n1-standard-1`, 1 cpu, 3.75 GB.

List available machines for selected zone:

```shell
gce machine-types list --filter="zone:(us-central1-a) AND guestCpus=4 AND memoryMb>=8000"
```

Machine types:

Types: [https://cloud.google.com/compute/docs/machine-types](https://cloud.google.com/compute/docs/machine-types).

Prices: [https://cloud.google.com/compute/vm-instance-pricing](https://cloud.google.com/compute/vm-instance-pricing)

| Type        | Purpose               |
| ----------- | --------------------- |
| E2          | Costs optimized       |
| N1, N2, N2D | Balanced              |
| Tau T2D     | Scale-out optimized   |
| M1, M2      | Memory-optimized      |
| C2, C2D     | Compute-optimized     |
| A2          | Accelerator-optimized |

Some common machines:

| Type          | vCPUs | RAM | Costs ($ / month) |
| ------------- | ----: | --: | ----------------: |
| e2-micro \*   |  0.25 |   1 |                 7 |
| e2-small \*   |   0.5 |   2 |                13 |
| e2-medium \*  |     1 |   4 |                25 |
| e2-standard-2 |     2 |   8 |                50 |
| e2-standard-4 |     4 |  16 |                98 |
| e2-standard-8 |     8 |  32 |               196 |
| e2-highcpu-2  |     2 |   2 |                37 |
| e2-highcpu-4  |     4 |   4 |                73 |
| n2-standard-2 |     2 |   8 |                57 |
| n2-standard-4 |     4 |  16 |               114 |
| n2-standard-8 |     8 |  32 |               228 |
| n2-highcpu-2  |     2 |   2 |                42 |
| n2-highcpu-4  |     4 |   4 |                84 |

\* `micro`, `small`, `medium` - is a machines with the shared vCPUs.

## Compute

### Set instance tags

```shell
gce instances add-tags test --tags=nginx
```

### List available OS images

Used for `--image-project` and `--image-family` options.

```shell
gce images list
```

## Services

### Enable service

```shell
# compute
gc services enable compute.googleapis.com

# maps
gc services enable places-backend.googleapis.com
gc services enable geocoding-backend.googleapis.com
gc services enable maps-backend.googleapis.com
gc services enable timezone-backend.googleapis.com
```

### Create API key

```shell
gc alpha services api-keys create --display-name=maps.local \
    --api-target=service=places-backend.googleapis.com \
    --api-target=service=geocoding-backend.googleapis.com \
    --allowed-referrers=localhost
```

### Get API key

```shell
gc alpha services api-keys get-key-string --format="get(keyString)" $(gc alpha services api-keys list --format="get(uid)" --filter=displayName=maps.local)
```

## Billing account management

### Add billling manager

Open resources management: [https://console.cloud.google.com/cloud-resource-manager?authuser=zdm@softvisio.net](https://console.cloud.google.com/cloud-resource-manager?authuser=zdm@softvisio.net).

Add new user with the following permissions: - `Owner`; - or `Viewer` + `Project Billing Manager`;

On the current and target Cloud Billing account: `Billing Account User` + `Billing Account Viewer` OR `Billing Account Administrator`.

### Rename billing account

Open billing accounts: [https://console.cloud.google.com/billing?authuser=zdm@softvisio.net](https://console.cloud.google.com/billing?authuser=zdm@softvisio.net).

Open required billing account.

At the top-left menu select `Account management`.

At the top press `Rename` button.
