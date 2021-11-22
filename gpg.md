# GPG

### Generate gpg key pairs

```shell
gpg --full-gen-key
```

### Generate revocation certificate

```shell
gpg --output revoke.asc --gen-revoke zdm@softvisio.net
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
