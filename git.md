# Git

### GitHub links

- Link to the docs site, returns data with the correct `Content-Type` header:

```sh
curl https://${OWNER}.github.io/${NAME}/${PATH}
```

- Link to the content from the repository. Returns data **without** `Content-Type` header:

```sh
curl https://raw.githubusercontent.com/${OWNER}/${NAME}/${BRANCH}/${PATH}
```

- Download file from the private repository:

```sh
curl https://${TOKEN}@raw.githubusercontent.com/${OWNER}/${NAME}/${BRANCH}/${PATH}
```

- Link to the `LFS` repository content:

```sh
curl https://media.githubusercontent.com/media/${OWNER}/${NAME}/${BRANCH}/${PATH}
```

- Download release asset:

```sh
curl https://github.com/${OWNER}/${NAME}/releases/download/${TAG}/${FILE_NAME}
```

- Download branch as archive:

```sh
curl https://github.com/${OWNER}/${NAME}/archive/${BRANCH}.tar.gz
```

- Download tag as archive:

```sh
curl https://github.com/${OWNER}/${NAME}/archive/refs/tags/${TAG}.tar.gz
```

### Move repository

```sh
git remote rm origin
git remote add origin $URL
git push origin --all
git push --tags
```

### Rename branch

- Rename local branch:

    ```sh
    git branch -m old new
    ```

- Rename remote branch:

    ```sh
    # rename remote branch
    git push origin :old new

    # update local branch upstream
    git branch new -u origin/new
    ```

### Restore deleted file

```sh
git checkout $(git rev-list -n 1 HEAD -- "$FILE_PATH")^ -- "$FILE_PATH"
```

### Remove untracked files

```sh
# files
git clean --force

# directories
git clean --force -d
```

### Delete commited changesets

```sh
git reset --hard $SHA1_COMMIT_ID
```

If changesets were pushed:

```sh
git push origin HEAD --force
```

### Delete file from history

```sh
git filter-repo --force --partial --invert-paths --path-match $FILE_PATH

git reflog expire --expire-unreachable=now --all
git gc --prune=now --aggressive
```

### Chnod

Mode: `XXXYYY`

where:

- `XXX`:

    - `100` - regular file
    - `120` - symlink

- `YYY`:

    - `644` - `rw-r--r--`
    - `755` - `rwxr-xr-x`

```sh
# list execulable files
git ls-files --format "%(objectmode) %(path)" | grep "^100755"

# make file in "bin" and "tests" directories executable
git ls-files -z bin tests | xargs -0 git update-index --chmod=+x

# make ".sh" file executable
git ls-files -z *.sh | xargs -0 git update-index --chmod=+x
```

### Encryption with `git-crypt`

```sh
# install
apt install --yes git-crypt

# init
git crypt init

# symmetric key
git crypt export-key secret.git-crypto
git crypt unlock secret.git-crypto

# gpg
git crypt add-gpg-user --trusted $EMAIL
git crypt unlock
```

### Remove all commits, except last

```sh
git reset --soft $(git rev-list --max-parents=0 HEAD)
git commit -a --amend -m "chore: init"
git push origin HEAD --force

git reflog expire --expire-unreachable=now --all
git gc --prune=now --aggressive
```

Sync other clone with updates:

```sh
git fetch --all
git reset --hard origin/main

git reflog expire --expire-unreachable=now --all
git gc --prune=now --aggressive
```

### Manage remotes

```sh
# create remote "upstream"
git remote add upstream --no-mirror git@github.com:${OWNER}/${NAME}.git

# track "main" branch only
git remote set-branches upstream main

# add tracking branch
git remote set-branches --add upstream $BRANCH_NAME
```

### Sync fork

- Current repository:

    ```sh
    # sync local "main" branch with the upstream "main" brach
    gh repo sync

    # sync local "main" branch with the upstream "feat" branch
    gh repo sync --branch feat
    ```

- Remote repository:

    ```sh
    # sync remote "main" branch with the upstream "feat" branch
    gh repo sync softvisio/test --branch feat
    ```

- Manual sync:

    ```sh
    # fetch changes from the upstream
    git fetch upstream

    # merge "main" branch with upstream
    git sw main
    git merge upstream/main
    ```
