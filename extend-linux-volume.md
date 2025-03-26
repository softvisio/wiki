# Extend linux volume

```sh
vgdisplay
```

Look at `Free PE / Size` to see how much free space you have.

### Resize

```sh
lvextend /dev/mapper/fedora_devel-root -L +30G

fsadm resize /dev/mapper/fedora_devel-root

df -h
```
