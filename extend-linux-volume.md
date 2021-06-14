# Extend linux volume

```shell
vgdisplay
```

Look at `Free PE / Size` to see how much free space you have.

### Resize

```shell
lvextend /dev/mapper/fedora_devel-root -L +30G

fsadm resize /dev/mapper/fedora_devel-root

df -h
```
