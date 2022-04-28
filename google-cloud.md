# Google cloud

Links:

-   [Cloud DNS](https://cloud.google.com/compute/docs/internal-dns)
-   [Cloud IP addresses](https://cloud.google.com/compute/docs/ip-addresses)
-   [Cloud init](https://www.digitalocean.com/community/tutorials/an-introduction-to-cloud-config-scripting)

NOTE:

-   `gc` is the alias for `gcloud`;
-   `gcc` is the alias for `gcloud config configurations`;
-   `gce` is the alias for `gcloud compute`;

### Init gcloud

```shell
gc init --console-only
```

### Create project

Project id must be unique across all google cloud platform. So use project id in `<organization-name>-<project-name>` format.

Create project:

```shell
gc projects create <project-id> --name <project-name>
```

Link billing account:

```shell
gc billing accounts list

gc billing projects link <project-id> --billing-account <billing-account-id>
```

After project is created you need to create project configuration file at `~/.config/gcloud/configurations/config_<project_name>`.

Set project region and zone according to the ping latency value [https://www.gcping.com](https://www.gcping.com).

Activate project configuration:

```shell
gcc activate <prohect-name>
```

### Create project cluster

Reserve static ip address for cluster entry point:

```shell
gce addresses create nginx
```

Setup firewall:

```shell
gce firewall-rules delete default-allow-rdp

gce firewall-rules update default-allow-http --target-tags=nginx
gcloud compute firewall-rules update default-allow-https --target-tags=nginx
```

```shell
gce firewall-rules create allow-http \
    --allow=tcp:80 \
    --direction=INGRESS \
    --source-ranges=0.0.0.0/0 \
    --target-tags=nginx
```

```shell
gce firewall-rules create allow-pgsql \
    --allow=tcp:5432 \
    --direction=INGRESS \
    --source-ranges=0.0.0.0/0 \
    --target-tags=nginx
```

```shell
gce firewall-rules create allow-softvisio-proxy \
    --allow=tcp:51930 \
    --direction=INGRESS \
    --source-ranges=0.0.0.0/0 \
    --target-tags=nginx
```

Create entry point instance:

```shell
gce instances create test \
    --image-project=ubuntu-os-cloud --image-family=ubuntu-minimal-2204-lts \
    --machine-type=e2-standard-4 \
    --metadata-from-file user-data=cloud-init.yaml \
    --address=nginx \
    --tags=nginx
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

`micro`, `small`, `medium` - is a machines with the shared vCPU.

| Type        | Purpose               |
| ----------- | --------------------- |
| E2          | Costs optimized       |
| N1, N2, N2D | Balanced              |
| Tau T2D     | Scale-out optimized   |
| M1, M2      | Memory-optimized      |
| C2, C2D     | Compute-optimized     |
| A2          | Accelerator-optimized |

### Compute

#### Create instance

Links:

-   [Create instance](https://cloud.google.com/container-optimized-os/docs/how-to/create-configure-instance)

```shell
gcloud compute instances create test \
    --image-project=ubuntu-os-cloud \
    --image-family=ubuntu-minimal-2204-lts \
    --machine-type=n1-standard-1 \
    --metadata-from-file user-data=cloud-init.yaml \
    --tags=nginx \
    --address=nginx # use reserved external addresss
# --network-interface=no-address # withoout external address
```

#### Set instance tags

```shell
gcloud compute instances add-tags test \
    --zone=us-central1-a \
    --tags=nginx
```

#### List available OS images

Used for `--image-project` and `--image-family` options.

```shell
gcloud compute images list
```

#### List available machines types

Default type is: `n1-standard-1`, 1 cpu, 3.75 GB

```shell
gcloud compute machine-types list --filter="zone:(us-central1-a) AND guestCpus=4 AND memoryMb>=8000"
```

#### Connect to the instance using SSH

```shell
gcloud compute ssh --zone=us-central1-a test
```

### Addresses

#### Reserve external address

```shell
gcloud compute addresses create nginx \
    --region=us-central1 \
    --addresses=35.202.255.145
```

### Services

#### Enable service

```shell
gcloud services enable artifactregistry.googleapis.com
```
