# Google cloud

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
