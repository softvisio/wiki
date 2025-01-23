# GPG

### Generate gpg key pairs

```shell
gpg --full-gen-key
```

### Authenticate with gpg

You need to create sub-key with the authentication capability:

```shell
gpg --expert --edit-key zdm@softvisio.net

addkey

# select ECC (set your own capabilities)
# follow instructions
```

### Export public ssh key

```shell
gpg --export-ssh-key zdm@softvisio.net
```

### Export private gpg key

```shell
gpg --armor --export-secret-keys zdm@softvisio.net
```

### Import gpg key (private or public)

```shell
gpg --import private.key
```

### Keyserver

Publish private key:

```shell
gpg --send-keys <KEY-ID>
```

Search / import keys:

```shell
gpg --search-keys <NAME-OR-EMAIL>
```

Refresh keys:

```shell
gpg --refresh-keys
```

### Revoke and delete private key

:warning: Do not delete private keys without revoke them before.

Revocation certificate is created automatically on private key generation.

1. Find certificate related to the private key.

2. Remove ":" at ":-----BEGIN PGP PUBLIC KEY BLOCK-----".

3. Revoke:

```shell
gpg --import openpgp-revocs.d/ < KEY-ID > .rev
```

Or you can manually generate revocation certificate:

```shell
# create revocation certificate
gpg --gen-revoke --output revoke.asc <KEY-ID>

# revoke private key
gpg --import revoke.asc

# cleanup
rm -f revoke.asc
```

Send revoked key to the `keyserver`:

```shell
gpg --send-keys <KEY-ID>
```

Delete revoked private key from the local storage:

```shell
gpg --delete-secret-key <KEY-ID>
```

### Delete public key

```shell
gpg --delete-key <KEY-ID>
```
