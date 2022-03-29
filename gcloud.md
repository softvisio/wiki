# Google cloud

[Cloud DNS](https://cloud.google.com/compute/docs/internal-dns)

[Cloud IP addresses](https://cloud.google.com/compute/docs/ip-addresses)

[Cloud init](https://www.digitalocean.com/community/tutorials/an-introduction-to-cloud-config-scripting)

### List available OS images

Used for `--image-family` and `--image-project` options.

```shell
gcloud compute images list
```

### List available machines types

Default type is: `n1-standard-1`, 1 cpu, 3.75 GB

```shell
gcloud compute machine-types list --filter="zone:(us-central1-a) AND guestCpus=4 AND memoryMb>=8000"
```

### List instances

```shell
gcloud compute instances list
```

### Delete instance

```shell
gcloud compute instances delete --zone=us-central1-a <INSTANCE-NAME>
```

### SSH

```shell
gcloud compute ssh --zone=us-central1-a <INSTANCE-NAME>
```

### Create instance

[Documentation](https://cloud.google.com/container-optimized-os/docs/how-to/create-configure-instance#gcloud_1)

#### Ubuntu

```shell
gcloud compute instances create test \
    --image-family=ubuntu-minimal-2004-lts \
    --image-project=ubuntu-os-cloud \
    --zone=us-central1-a \
    --machine-type=c2d-highcpu-4 \
    --metadata-from-file user-data=cloud-init.ubuntu.yaml \
    --tags TAG1,TAG2 \
    --address=load-balancer
```

#### Without external IP address

```shell
--network-interface=no-address
```

### Add instance tags

```shell
gcloud compute instances add-tags test \
    --zone us-central1-a \
    --tags http-server
```

### Allow port

```shell
gcloud compute firewall-rules create allow-pgsql \
    --description="Allow incoming traffic on TCP port 5432" \
    --allow=tcp:5432 \
    --direction=INGRESS \
    --source-ranges=0.0.0.0/0 \
    --target-tags=http-server
```

### Reserve external address

```shell
gcloud compute addresses create load-balancer \
    --region=us-central1 \
    --addresses=35.202.255.145
```

### Enable service

```shell
gcloud services enable artifactregistry.googleapis.com
```
