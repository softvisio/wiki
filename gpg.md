# GPG

### Environment

Create temporary `GPG` home:

```sh
export GNUPGHOME="$(mktemp -d)"
```

### Generate key

Interactive:

```sh
gpg --full-gen-key
```

Automated:

<https://www.gnupg.org/documentation/manuals/gnupg/Unattended-GPG-key-generation.html>

Key types / curves:

- `EdDSA`: `Ed25519`
- `ECDSA`: `NIST P-256`, `NIST P-384`, `NIST P-521`
- `ECC`: `Cv25519`

```sh
gpg --batch --generate-key << EOF
    Key-Type: EdDSA
    Key-Curve: Ed25519
    Key-Usage: cert
    Subkey-Type: EdDSA
    Subkey-Curve: Ed25519
    Subkey-Usage: sign
    Name-Email: $EMAIL
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

```sh
gpg --export-secret-key --armor --output private-key.asc $KEY_ID
```

Export `public` key:

```sh
gpg --export --armor --output public-key.asc $KEY_ID
```

Export `SSH public` key:

```sh
gpg --export-ssh-key $KEY_ID
```

### Import keys

```sh
# import unencrypted key
gpg --import $KEY_PATH

# import key from url
curl https://raw.githubusercontent.com/zdm/secure/main/backup/deployment@softvisio.net.asc \
    | gpg --import

# import encrypted key
gpg --decrypt --batch --passphrase $PASSWORD $KEY_PATH \
    | gpg --import

# import encrypted key from url
export GPG_TTY=$(tty)
curl https://raw.githubusercontent.com/zdm/secure/main/backup/deployment@softvisio.net.asc \
    | gpg --decrypt --batch --passphrase $PASSWORD \
    | gpg --import
```

### Backup / restore

```sh
# full backup
gpg --export-secret-keys \
    | gpg --symmetric --batch --armor --yes --passphrase $PASSWORD --output gpg-backup.asc

gpg --export-ownertrust > gpg-ownertrust.txt

# restore
gpg --decrypt --batch --passphrase $PASSWORD gpg-backup.asc \
    | gpg --import --import-options restore

gpg --import-ownertrust gpg-ownertrust.txt
```

### SSH authentication

Generate sub-key with the `authentication` capability:

```sh
gpg --edit-key $KEY_ID

addkey

# select ECC (set your own capabilities)
# follow instructions
```

Add keygrip for key with `authenticate` capability to `sshcontrol` file:

```sh
gpg --list-keys --with-keygrip $KEY_ID
```

### Keyserver (publish keys)

Publish private key:

```sh
gpg --send-keys $KEY_ID
```

Search / import keys:

```sh
gpg --search-keys $NAME_OR_EMAIL
```

Refresh keys:

```sh
gpg --refresh-keys
```

Check supported schemes:

```sh
dirmngr

KEYSERVER --help
```

### Revoke and delete private key

:warning: Do not delete private keys without revoke them before.

Revocation certificate is created automatically on private key generation.

1. Find certificate related to the private key. Revocation certificate file name is equal to the private key full fingerprint: `openpgp-revocs.d/$FINGERPRINT.rev`.

2. Remove `":"` character from the `":-----BEGIN PGP PUBLIC KEY BLOCK-----"` line.

3. Revoke:

```sh
gpg --import "openpgp-revocs.d/$KEY_ID.rev"
```

Or you can manually generate revocation certificate:

```sh
# create revocation certificate
gpg --gen-revoke --output certificate.rev $KEY_ID

# revoke private key
gpg --import certificate.rev

# cleanup
rm -f certificate.rev
```

Send revoked key to the keyserver:

```sh
gpg --send-keys $KEY_ID
```

Delete revoked private key from the local keyring:

```sh
gpg --delete-secret-and-public-keys $KEY_ID
```

### Delete public key

```sh
gpg --delete-key $KEY_ID
```
