# GPG

### Environment

Create temporary `GPG` home:

```shell
export GNUPGHOME="$(mktemp -d)"
```

### Generate key

Interactive:

```shell
gpg --full-gen-key
```

Automated:

<https://www.gnupg.org/documentation/manuals/gnupg/Unattended-GPG-key-generation.html>

Key types / curves:

- `EdDSA`: `Ed25519`
- `ECDSA`: `NIST P-256`, `NIST P-384`, `NIST P-521`

```shell
gpg --batch --generate-key << EOF
    Key-Type: EdDSA
    Key-Curve: Ed25519
    Key-Usage: cert
    Subkey-Type: EdDSA
    Subkey-Curve: Ed25519
    Subkey-Usage: sign
    Name-Email: <EMAIL>
    # Name-Real:
    # Name-Comment:
    Expire-Date: 0
    Keyserver: hkps://keyserver.ubuntu.com
    %no-protection
    %commit
EOF
```

### Export keys

Export `private` key:

```shell
gpg --export-secret-key --armor --output private-key.asc <KEY-ID>
```

Export `public` key:

```shell
gpg --export --armor --output public-key.asc <KEY-ID>
```

Export `SSH public` key:

```shell
gpg --export-ssh-key <KEY-ID>
```

### Import keys

```shell
gpg --import <PRIVATE-OR-PUBLIC-KEY-PATH>
```

### SSH authentication

Generate sub-key with the `authentication` capability:

```shell
gpg --edit-key <KEY-ID>

addkey

# select ECC (set your own capabilities)
# follow instructions
```

Add keygrip for key with `authenticate` capability to `sshcontrol` file:

```shell
gpg --list-keys --with-keygrip <KEY-ID>
```

### Keyserver (publish keys)

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

Check supported schemes:

```shell
dirmngr

KEYSERVER --help
```

### Revoke and delete private key

:warning: Do not delete private keys without revoke them before.

Revocation certificate is created automatically on private key generation.

1. Find certificate related to the private key. Revocation certificate file name is equal to the private key full fingerprint: `openpgp-revocs.d/<FINGERPRINT>.rev`.

2. Remove `":"` character from the `":-----BEGIN PGP PUBLIC KEY BLOCK-----"` line.

3. Revoke:

```shell
gpg --import "openpgp-revocs.d/<KEY-ID>.rev"
```

Or you can manually generate revocation certificate:

```shell
# create revocation certificate
gpg --gen-revoke --output certificate.rev <KEY-ID>

# revoke private key
gpg --import certificate.rev

# cleanup
rm -f certificate.rev
```

Send revoked key to the keyserver:

```shell
gpg --send-keys <KEY-ID>
```

Delete revoked private key from the local keyring:

```shell
gpg --delete-secret-and-public-keys <KEY-ID>
```

### Delete public key

```shell
gpg --delete-key <KEY-ID>
```
