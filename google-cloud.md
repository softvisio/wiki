# Google cloud

Links:

-   [Cloud DNS](https://cloud.google.com/compute/docs/internal-dns)
-   [Cloud IP addresses](https://cloud.google.com/compute/docs/ip-addresses)
-   [Cloud init](https://www.digitalocean.com/community/tutorials/an-introduction-to-cloud-config-scripting)

NOTE:

-   `gcp` is the alias for `gcloud config configurations`;
-   `gce` is the alias for `gcloud compute`;

### Init gcloud

```shell
gcloud init --console-only
```

### Create project

Project id must be unique across all google cloud platform. So use project id in `<organization-name>-<project-name>` format.

Create project:

```shell
gcloud projects create <project-id>
```

Link billing account:

```shell
gcloud billing accounts list

gcloud beta billing projects link <project-id> --billing-account <billing-account-id>
```

After beta project is created you need to create project configuration file at `~/.config/gcloud/configurations/config_<project_name>`.

Set project region and zone according to the ping latency value [https://www.gcping.com](https://www.gcping.com).

Activate project configuration:

```shell
gcp activate <project-name>
```

### Initialize clustes

```shell
# create default ssl policies
gce ssl-policies create compatible --profile=COMPATIBLE
gce ssl-policies create modern --profile=MODERN
gce ssl-policies create restricted --profile=RESTRICTED

# create default healthchecks
gce health-checks create tcp tcp --use-serving-port
gce health-checks create http http --use-serving-port --request-path=/healthcheck

# disable rdp
gce firewall-rules delete default-allow-rdp

# allow traffic from google load balancers and health checkers
gce firewall-rules create allow-load-balancer --source-ranges=130.211.0.0/22,35.191.0.0/16 --action=allow --rules=tcp,udp

# reserve ip addresses for load balancer
gce addresses create ipv4 --ip-version=IPV4 --global
gce addresses create ipv6 --ip-version=IPV6 --global
```

### Create instances

```shell
gce instances create a0 \
    --machine-type=e2-micro \
    --image-project=ubuntu-os-cloud --image-family=ubuntu-minimal-2204-lts \
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

### HTTPS load balancer

Create instances group:

```shell
gce instance-groups unmanaged create http
gce instance-groups unmanaged add-instances http --instances=a0
gce instance-groups set-named-ports main --named-ports tcp80:80
```

Create load balancer:

```shell
# create backend service
gce backend-services create http --global-health-checks --health-checks=tcp --protocol=http --port-name=tcp80 --global --cache-mode=USE_ORIGIN_HEADERS --enable-cdn

# add instances group to the backend service
gce backend-services add-backend http --instance-group=http --global

# create url map
gce url-maps create http --default-service=http

# create https proxy
gce target-https-proxies create https --ssl-certificates=certificate --ssl-policy=modern --url-map=http

# create forwarding rule
gce forwarding-rules create https --load-balancing-scheme=external --ip-protocol=tcp --address=ipv4 --ports=443 --target-https-proxy=https --global
```

Remove load balancer:

```shell
yes | gce forwarding-rules delete https --global
yes | gce target-https-proxies delete https
yes | gce url-maps delete http
yes | gce backend-services delete http --global
```

### HTTP SSL load balancer

Create instances group:

```shell
gce instance-groups unmanaged create http
gce instance-groups unmanaged add-instances http --instances=a0
gce instance-groups set-named-ports main --named-ports tcp80:80
```

Create load balancer:

```shell
# create backend service
gce backend-services create http --global-health-checks --health-checks=tcp --protocol=tcp --port-name=tcp80 --global

# add instances group to the backend service
gce backend-services add-backend http --instance-group=http --global

# create https proxy
gce target-ssl-proxies create https --backend-service=http --ssl-certificates=certificate --proxy-header=PROXY_V1 --ssl-policy=modern

# create forwarding rule
gce forwarding-rules create https --load-balancing-scheme=external --ip-protocol=tcp --address=ipv4 --ports=443 --target-ssl-proxy=https --global
```

Remove load balancer:

```shell
yes | gce forwarding-rules delete https --global
yes | gce target-ssl-proxies delete https
yes | gce backend-services delete http --global
```

### PostgreSQL SSL load balancer

Create instances group:

```shell
gce instance-groups unmanaged create pgsql
gce instance-groups unmanaged add-instances pgsql --instances=a0
gce instance-groups set-named-ports pgsql --named-ports tcp5432:5432
```

Create load balancer:

```shell
# create backend service
gce backend-services create pgsql --global-health-checks --health-checks=tcp --protocol=tcp --port-name=tcp5432 --global

# add instances group to the backend service
gce backend-services add-backend pgsql --instance-group=pgsql --global

# create ssl proxy
gce target-ssl-proxies create pgsql --backend-service=pgsql --ssl-certificates=certificate --ssl-policy=restricted

# create forwarding rule
gce forwarding-rules create pgsql --load-balancing-scheme=external --ip-protocol=tcp --address=ipv4 --ports=5432 --target-ssl-proxy=pgsql --global
```

Remove load balancer:

```shell
yes | gce forwarding-rules delete pgsql --global
yes | gce target-ssl-proxies delete pgsql
yes | gce backend-services delete pgsql --global
```

### PostgreSQL TCP load balancer

Create instances group:

```shell
gce instance-groups unmanaged create pgsql
gce instance-groups unmanaged add-instances pgsql --instances=a0
gce instance-groups set-named-ports pgsql --named-ports tcp5432:5432
```

Create load balancer:

```shell
# create backend service
gce backend-services create pgsql --global-health-checks --health-checks=tcp --protocol=tcp --port-name=tcp5432 --global

# add instances group to the backend service
gce backend-services add-backend pgsql --instance-group=pgsql --global

# create ssl proxy
gce target-tcp-proxies create pgsql --backend-service=pgsql

# create forwarding rule
gce forwarding-rules create pgsql --load-balancing-scheme=external --ip-protocol=tcp --address=ipv4 --ports=5432 --target-tcp-proxy=pgsql --global
```

Remove load balancer:

```shell
yes | gce forwarding-rules delete pgsql --global
yes | gce target-tcp-proxies delete pgsql
yes | gce backend-services delete pgsql --global
```

### Create project cluster

Reserve regional static ip address for cluster entry point:

```shell
gce addresses create tcp-load-balancer
```

Create target pool:

```shell
gce target-pools create tcp-load-balancer
```

Add instances to the target pool:

```shell
gce target-pools add-instances tcp-load-balancer --instances=a0
```

Create load balancer forwarding rule:

```shell
gce forwarding-rules create tcp-load-balancer --target-pool=tcp-load-balancer --load-balancing-scheme=EXTERNAL --ports=1-65535 --address=tcp-load-balancer --ip-protocol=TCP
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

#### Connect to the instance using SSH

```shell
gce ssh test
```

### Services

#### Enable service

```shell
gcloud services enable artifactregistry.googleapis.com
```
