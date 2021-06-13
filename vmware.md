# Compact `.vmdk`

```batch
# fill free space with zeros
cat /dev/zero >/wipefile
rm -rf /wipefile
```

```batch
:: defragment all .vmdk files in the current directory
for /R %f in (*.vmdk) do vmware-vdiskmanager -d "%f"

:: shrink all .vmdk files in the current directory
for /R %f in (*.vmdk) do vmware-vdiskmanager -k "%f"
```
