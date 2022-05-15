# Google cloud

Links:

-   [Cloud DNS](https://cloud.google.com/compute/docs/internal-dns)
-   [Cloud IP addresses](https://cloud.google.com/compute/docs/ip-addresses)
-   [Cloud init](https://www.digitalocean.com/community/tutorials/an-introduction-to-cloud-config-scripting)

NOTE:

-   `gc` is the alias for `gcloud`;
-   `gce` is the alias for `gcloud compute`;
-   `gcp` is the alias for `gcloud config configurations`;

### Init gcloud

```shell
gc init --console-only
```

### Create project

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

### Initialize cluster

```shell
# create default ssl policies
gce ssl-policies create compatible --profile=COMPATIBLE
gce ssl-policies create modern --profile=MODERN
gce ssl-policies create restricted --profile=RESTRICTED

# create default healthchecks
gce health-checks create tcp tcp --use-serving-port
gce health-checks create http http --use-serving-port --request-path=/healthcheck

# disable rdp
gce firewall-rules update default-allow-rdp --disabled

# allow traffic from google load balancers and health checkers
gce firewall-rules create allow-load-balancer --source-ranges=130.211.0.0/22,35.191.0.0/16 --action=ALLOW --rules=tcp,udp

# reserve ip addresses for load balancer
gce addresses create public-ipv4 --ip-version=IPV4 --global
gce addresses create private-ipv4 --ip-version=IPV4 --global

# create nat
gce routers create nat --network=default
gce routers nats create nat --router=nat --auto-allocate-nat-external-ips --nat-all-subnet-ip-ranges --enable-dynamic-port-allocation
```

### Create instances

```shell
gce instances create a0 \
    --machine-type=e2-micro \
    --image-project=ubuntu-os-cloud --image-family=ubuntu-minimal-2204-lts \
    --network-interface=no-address \
    --metadata-from-file user-data=cloud-init.yaml
```

### Create certificates

Before start:

-   Create `dns` records for all used domains, using load valancer ip addresses.
-   On `cloudflare` for each domain separately add CAA records for "pki.goog".

Create certificate:

```shell
gce ssl-certificates create certificate --domains=httpbin.softvisio.net --global
```

Check certificates status:

```shell
gce ssl-certificates list
```

### Create instances group

```shell
gce instance-groups unmanaged create nginx
gce instance-groups unmanaged add-instances nginx --instances=a0
gce instance-groups set-named-ports nginx --named-ports=tcp80:80,tcp5432:5432
```

### HTTP load balancer

Create redirect from `http` to `https`:

```shell
gce url-maps import http --global --source=http.url-map.yaml

gce target-http-proxies create http --url-map=http --global

gce forwarding-rules create http --load-balancing-scheme=EXTERNAL --address=public-ipv4 --ports=80 --target-http-proxy=http --global
```

Remove load balancer:

```shell
yes | gce forwarding-rules delete http --global
yes | gce target-http-proxies delete http
yes | gce url-maps delete http
```

### HTTPS load balancer

Create load balancer:

```shell
# create backend service
gce backend-services create http --global-health-checks --health-checks=tcp --protocol=HTTP --port-name=tcp80 --global --custom-response-header="Strict-Transport-Security:max-age=63072000; includeSubdomains; preload" # --cache-mode=USE_ORIGIN_HEADERS --enable-cdn

# add instances group to the backend service
gce backend-services add-backend http --global --instance-group=nginx

# create url map
gce url-maps create https --default-service=http

# create https proxy
gce target-https-proxies create https --ssl-certificates=certificate --ssl-policy=modern --url-map=https

# create forwarding rule
gce forwarding-rules create https --load-balancing-scheme=EXTERNAL --address=public-ipv4 --ports=443 --target-https-proxy=https --global
```

Remove load balancer:

```shell
yes | gce forwarding-rules delete https --global
yes | gce target-https-proxies delete https
yes | gce url-maps delete https
yes | gce backend-services delete http --global
```

### PostgreSQL SSL load balancer

SSL load balancer not works with the `psql` client. Use firewall rule for the specific instance until this will be solved:

```shell
gce firewall-rules create allow-pgsql --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:5432
```

Create load balancer:

```shell
# create backend service
gce backend-services create pgsql --global-health-checks --health-checks=tcp --protocol=TCP --port-name=tcp5432 --global

# add instances group to the backend service
gce backend-services add-backend pgsql --global --instance-group=nginx

# create ssl proxy
gce target-ssl-proxies create pgsql --backend-service=pgsql --ssl-certificates=certificate --ssl-policy=restricted

# create forwarding rule
gce forwarding-rules create pgsql --load-balancing-scheme=EXTERNAL --address=private-ipv4 --ports=5432 --target-ssl-proxy=pgsql --global
```

Remove load balancer:

```shell
yes | gce forwarding-rules delete pgsql --global
yes | gce target-ssl-proxies delete pgsql
yes | gce backend-services delete pgsql --global
```

### Machine type

Default type is: `n1-standard-1`, 1 cpu, 3.75 GB.

List available machines for selected zone:

```shell
gce machine-types list --filter="zone:(us-central1-a) AND guestCpus=4 AND memoryMb>=8000"
```

Machine types:

Types: [https://cloud.google.com/compute/docs/machine-types](https://cloud.google.com/compute/docs/machine-types).

Prices: [https://cloud.google.com/compute/vm-instance-pricing](https://cloud.google.com/compute/vm-instance-pricing)

`micro`, `small`, `medium` - is a machines with the shared vCPUs.

| Type        | Purpose               |
| ----------- | --------------------- |
| E2          | Costs optimized       |
| N1, N2, N2D | Balanced              |
| Tau T2D     | Scale-out optimized   |
| M1, M2      | Memory-optimized      |
| C2, C2D     | Compute-optimized     |
| A2          | Accelerator-optimized |

Some common machines:

| Type          | vCPUs | RAM | Costs ($) |
| ------------- | ----: | --: | --------: |
| e2-small      |     2 |   2 |        25 |
| e2-standard-2 |     2 |   8 |        50 |
| e2-standard-4 |     4 |  16 |        98 |
| e2-standard-8 |     8 |  32 |       196 |
| e2-highcpu-2  |     2 |   2 |        37 |
| e2-highcpu-4  |     4 |   4 |        73 |
| n2-standard-2 |     2 |   8 |        57 |
| n2-standard-4 |     4 |  16 |       114 |
| n2-standard-8 |     8 |  32 |       228 |
| n2-highcpu-2  |     2 |   2 |        42 |
| n2-highcpu-4  |     4 |   4 |        84 |

### Compute

#### Set instance tags

```shell
gce instances add-tags test --tags=nginx
```

#### List available OS images

Used for `--image-project` and `--image-family` options.

```shell
gce images list
```

### Services

#### Enable service

```shell
gc services enable artifactregistry.googleapis.com
```
