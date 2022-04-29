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
gcloud projects create <project-id> --name <project-name>
```

Link billing account:

```shell
gcloud billing accounts list

gcloud billing projects link <project-id> --billing-account <billing-account-id>
```

After project is created you need to create project configuration file at `~/.config/gcloud/configurations/config_<project_name>`.

Set project region and zone according to the ping latency value [https://www.gcping.com](https://www.gcping.com).

Activate project configuration:

```shell
gcp activate <project-name>
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
gce firewall-rules update default-allow-https --target-tags=nginx
```

Open `http` port:

```shell
gce firewall-rules create allow-http \
    --allow=tcp:80 \
    --direction=INGRESS \
    --source-ranges=0.0.0.0/0 \
    --target-tags=nginx
```

Open `PostgreSQL` port:

```shell
gce firewall-rules create allow-pgsql \
    --allow=tcp:5432 \
    --direction=INGRESS \
    --source-ranges=0.0.0.0/0 \
    --target-tags=nginx
```

Open `proxy` service port:

```shell
gce firewall-rules create allow-softvisio-proxy \
    --allow=tcp:51930 \
    --direction=INGRESS \
    --source-ranges=0.0.0.0/0 \
    --target-tags=nginx
```

Create entry point instance:

```shell
gce instances create a0 \
    --machine-type=e2-standard-2 \
    --image-project=ubuntu-os-cloud --image-family=ubuntu-minimal-2204-lts \
    --metadata-from-file user-data=cloud-init.yaml \
    --tags=nginx \
    --address=nginx
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
