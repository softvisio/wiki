# Google cloud

Links:

- [Cloud DNS](https://cloud.google.com/compute/docs/internal-dns)
- [Cloud IP addresses](https://cloud.google.com/compute/docs/ip-addresses)
- [Cloud init](https://www.digitalocean.com/community/tutorials/an-introduction-to-cloud-config-scripting)

NOTE:

- `gc` is the alias for `gcloud`;
- `gce` is the alias for `gcloud compute`;
- `gcp` is the alias for `gcloud config configurations`;

## Init gcloud

```sh
gc init --console-only
```

## Create project

Project id must be unique across all google cloud platform. So use project id in `<ORGANIZATION-NAME>-<PROJECT-NAME>` format.

Create project:

```sh
gc projects create $PROJECT_ID
```

Enabled GCE API : `https://console.developers.google.com/apis/api/compute.googleapis.com/overview?authuser=dzagashev@gmail.com&project=<PROJECT-ID>`

Link billing account:

```sh
gc beta billing accounts list

gc beta billing projects link $PROJECT_ID --billing-account $BILLING_ACCOUNT_ID
```

After project is created you need to create project configuration file at `~/.config/gcloud/configurations/config_$PROJECT_NAME`.

Set project region and zone according to the ping latency value <https://www.gcping.com>.

Activate project configuration:

```sh
gcp activate $PROJECT_NAME
```

## Initialize cluster

```sh
# disable default firewall rules
gce firewall-rules update default-allow-rdp --disabled
gce firewall-rules update default-allow-icmp --disabled

# allow IPv4 HTTP traffic to load balancer
yes | gce firewall-rules delete allow-ipv4-http-to-load-balancer
gce firewall-rules create allow-ipv4-http-to-load-balancer \
    --description="Allow IPv4 HTTP traffic to load balancer" \
    --action=ALLOW \
    --rules=tcp:80,tcp:443,udp:443 \
    --target-tags=load-balancer

# allow IPv4 HTTP traffic from CloudFlare to load balancer
yes | gce firewall-rules delete allow-ipv4-http-cloudflare-to-load-balancer
gce firewall-rules create allow-ipv4-http-cloudflare-to-load-balancer \
    --description="Allow IPv4 HTTP traffic from CloudFlare to load balancer" \
    --action=ALLOW \
    --source-ranges=$(curl -fsSL https://www.cloudflare.com/ips-v4 | xargs | sed -e "s/ /,/g") \
    --rules=tcp:80,tcp:443,udp:443 \
    --target-tags=load-balancer

# allow IPv6 HTTP traffic from CloudFlare to load balancer
yes | gce firewall-rules delete allow-ipv6-http-cloudflare-to-load-balancer
gce firewall-rules create allow-ipv6-http-cloudflare-to-load-balancer \
    --description="Allow IPv6 HTTP traffic from CloudFlare to load balancer" \
    --action=ALLOW \
    --source-ranges=$(curl -fsSL https://www.cloudflare.com/ips-v6 | xargs | sed -e "s/ /,/g") \
    --rules=tcp:80,tcp:443,udp:443 \
    --target-tags=load-balancer

# allow IPv4 tcp:8085 traffic to load balancer
yes | gce firewall-rules delete allow-ipv4-8085-to-load-balancer
gce firewall-rules create allow-ipv4-8085-to-load-balancer \
    --description="Allow IPv4 tcp:8085 traffic to load balancer" \
    --action=ALLOW \
    --rules=tcp:8085 \
    --target-tags=load-balancer
```

## Create instances

Create `load-balancer` instance:

```sh
# reserve regional ip address for load balancer instance
gce addresses create public-ipv4

gce instances create a0 \
    --machine-type=e2-micro \
    --image-project=ubuntu-os-cloud --image-family=ubuntu-minimal-2604-lts-amd64 \
    --address=public-ipv4 \
    --metadata-from-file user-data=cloud-init.sh \
    --tags=load-balancer
```

Create `worker` instance:

```sh
gce instances create b0 \
    --machine-type=e2-micro \
    --image-project=ubuntu-os-cloud --image-family=ubuntu-minimal-2604-lts-amd64 \
    --metadata-from-file user-data=cloud-init.sh \
    --metadata init_docker="docker swarm join --token ..."
```

## Machine type

Default type is: `n1-standard-1`, 1 cpu, 3.75 GB.

List available machines for selected zone:

```sh
gce machine-types list --filter="zone:(us-central1-a) AND guestCpus=4 AND memoryMb>=8000"
```

Machine types:

Types: <https://cloud.google.com/compute/docs/machine-types>.

Prices: <https://cloud.google.com/compute/vm-instance-pricing>

| Type        | Purpose               |
| ----------- | --------------------- |
| E2          | Costs optimized       |
| N1, N2, N2D | Balanced              |
| Tau T2D     | Scale-out optimized   |
| M1, M2      | Memory-optimized      |
| C2, C2D     | Compute-optimized     |
| A2          | Accelerator-optimized |

Some common machines:

| Type           | vCPUs | RAM | Costs ($ / month) |
| -------------- | ----: | --: | ----------------: |
| e2-micro [^1]  |  0.25 |   1 |                 7 |
| e2-small [^1]  |   0.5 |   2 |                13 |
| e2-medium [^1] |     1 |   4 |                25 |
| e2-standard-2  |     2 |   8 |                50 |
| e2-standard-4  |     4 |  16 |                98 |
| e2-standard-8  |     8 |  32 |               196 |
| e2-highcpu-2   |     2 |   2 |                37 |
| e2-highcpu-4   |     4 |   4 |                73 |
| n2-standard-2  |     2 |   8 |                57 |
| n2-standard-4  |     4 |  16 |               114 |
| n2-standard-8  |     8 |  32 |               228 |
| n2-highcpu-2   |     2 |   2 |                42 |
| n2-highcpu-4   |     4 |   4 |                84 |

[^1]: `micro`, `small`, `medium` - is a machines with the shared vCPUs.

## Compute

### Set instance tags

```sh
gce instances add-tags test --tags=nginx
```

### List available OS images

Used for `--image-project` and `--image-family` options.

```sh
gce images list --project=ubuntu-os-cloud --filter="family ~ ubuntu-minimal"
```

## Services

### Enable service

```sh
# compute
gc services enable compute.googleapis.com

# maps
gc services enable places-backend.googleapis.com
gc services enable geocoding-backend.googleapis.com
gc services enable maps-backend.googleapis.com
gc services enable timezone-backend.googleapis.com
```

### Create API key

```sh
gc alpha services api-keys create --display-name=maps.local \
    --api-target=service=places-backend.googleapis.com \
    --api-target=service=geocoding-backend.googleapis.com \
    --allowed-referrers=localhost
```

### Get API key

```sh
gc alpha services api-keys get-key-string --format="get(keyString)" $(gc alpha services api-keys list --format="get(uid)" --filter=displayName=maps.local)
```

## Billing account management

### Add billing manager

Open resources management: <https://console.cloud.google.com/cloud-resource-manager?authuser=dzagashev@gmail.com>.

In the left-side panel add new user with the following roles: - `Viewer`; - `Project Billing Manager`;

Other roles, which can be useful:

- `Owner`;
- `Viewer` + `Billing Account Administrator`;
- `Billing Account User` + `Billing Account Viewer`;

### Change billing account

Oppen <https://console.cloud.google.com/billing/projects>.

Change billing account for the project.

### Rename billing account

Open billing accounts: <https://console.cloud.google.com/billing?authuser=dzagashev@gmail.com>.

Open needed billing account by clucking on it's nane.

At the top-left menu select `Account management`.

At the top press `Rename` button.

## OAuth access

### Gcloud OAuth

Go to <https://console.cloud.google.com/apis/credentials?authuser=dzagashev@gmail.com>.

Create OAuth client credentials:

URL: <https://domain>
Redirect URL: <https://domain/api/oauth.html>

Save credentials, update backend config witj clientId anf clientSecret.
