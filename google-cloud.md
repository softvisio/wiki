# Google cloud

[Cloud DNS](https://cloud.google.com/compute/docs/internal-dns)

[Cloud IP addresses](https://cloud.google.com/compute/docs/ip-addresses)

[Cloud init](https://www.digitalocean.com/community/tutorials/an-introduction-to-cloud-config-scripting)

### Compute

#### Create instance

[Documentation](https://cloud.google.com/container-optimized-os/docs/how-to/create-configure-instance#gcloud_1)

```shell
gcloud compute instances create test \
    --zone=us-central1-a \
    --image-family=ubuntu-minimal-2004-lts \
    --image-project=ubuntu-os-cloud \
    --machine-type=c2d-highcpu-4 \
    --metadata-from-file user-data=cloud-init.yaml \
    --tags TAG1,TAG2 \
    --address=nginx
# --network-interface=no-address # withoout external address
```

#### Set instance tags

```shell
gcloud compute instances add-tags test \
    --zone us-central1-a \
    --tags nginx
```

#### List available OS images

Used for `--image-family` and `--image-project` options.

```shell
gcloud compute images list
```

#### List available machines types

Default type is: `n1-standard-1`, 1 cpu, 3.75 GB

```shell
gcloud compute machine-types list --filter="zone:(us-central1-a) AND guestCpus=4 AND memoryMb>=8000"
```

#### List instances

```shell
gcloud compute instances list
```

#### Delete instance

```shell
gcloud compute instances delete --zone=us-central1-a <INSTANCE-NAME>
```

#### SSH

```shell
gcloud compute ssh --zone=us-central1-a <INSTANCE-NAME>
```

### Firewall

#### Open PostgreSQL port:

```shell
gcloud compute firewall-rules create allow-pgsql \
    --allow=tcp:5432 \
    --direction=INGRESS \
    --source-ranges=0.0.0.0/0 \
    --target-tags=nginx
```

#### Open softvisio proxy port:

```shell
gcloud compute firewall-rules create allow-softvisio-proxy \
    --allow=tcp:51930 \
    --direction=INGRESS \
    --source-ranges=0.0.0.0/0 \
    --target-tags=nginx
```

#### Set rules tags

```shell
gcloud compute firewall-rules update default-allow-http --target-tags nginx
gcloud compute firewall-rules update default-allow-https --target-tags nginx
```

### Reserve external address

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
