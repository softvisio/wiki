# Git

### GitHub links

- Link to the docs site, returns data with the valid content type:

```
https://<OWNER>.github.io/<NAME>/
```

- Link to the content from the repository. Returns data **without** content type:

```
https://raw.githubusercontent.com/<OWNER>/<NAME>/<BRANCH>/
```

- Link to the `LFS` repository content:

```
https://media.githubusercontent.com/media/<OWNER>/<NAME>/<BRANCH>/
```

- Download release asset:

```
https://github.com/<OWNER>/<NAME>/releases/download/<TAG>/<FILE-NAME>
```

- Download branch as archive:

```
https://github.com/<OWNER>/<NAME>/archive/<BRANCH>.tar.gz
```

- Download tag as archive:

```
https://github.com/<OWNER>/<NAME>/archive/refs/tags/<TAG>.tar.gz
```

### Move repository

```sh
git remote rm origin
git remote add origin $URL
git push origin --all
git push --tags
```

### Sync forked repository

```sh
git clone ssh://$YOUR_FORKED_REPO
cd $YOUR_FORKED_REPO
git remote add upstream https://$ORIGINAL_REPO
git fetch upstream
git pull upstream main
git push
```

### Rename branch

In the following example we rename `master` to `main`:

```sh
git sw master
git branch -m main
git push origin -u main
git push origin --delete master
```

On `GitHub` repository `settings/branches` switch default branch to the `main`.

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

### Delete file from commits

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
git-crypt init

# symmetric key
git-crypt export-key secret.git-crypto
git-crypt unlock secret.git-crypto

# gpg
git-crypt add-gpg-user --trusted $EMAIL
git-crypt unlock
```

### Remove all commits, except last

```sh
git reset --soft $(git rev-list --max-parents=0 HEAD)
git commit -a --amend -m "chore: init"
git push origin HEAD --force
git gc

# sync other clone with updates
git fetch --all
git reset --hard origin/main
git gc
```
