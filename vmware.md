# VMWare

### Screen resolution

Use this screen resolution for `windows` and `macos`: `1400 x 610`

### Compact `.vmdk`

```sh
# fill free space with zeros
cat /dev/zero > ~/wipefile
rm -rf ~/wipefile
```

```batch
:: defragment all .vmdk files in the current directory
for /R %f in (*.vmdk) do vmware-vdiskmanager -d "%f"

:: shrink all .vmdk files in the current directory
for /R %f in (*.vmdk) do vmware-vdiskmanager -k "%f"
```

### MacOS

#### Unlocker

Need to reinstall on every vmware update.

```sh
git clone git@github.com:DrDonk/unlocker.git

# from admin
call win-install.cmd
```

#### Install

- Use APFS case-sensitive FS;
- Unsubscribe from the BETA program, System Preferences -> Updates -> Details;
- Change resolution

```sh
sudo "/Library/Application Support/VMware Tools/vmware-resolutionSet" 1920 1080
```
