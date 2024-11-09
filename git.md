# Git

### GitHub links

-   Link to the docs site, returns data with the valid content type:

    ```text
    https://<OWNER>.github.io/<NAME>/
    ```

-   Link to the content from the repository. Returns data **without** content type:

    ```text
    https://raw.githubusercontent.com/<OWNER>/<NAME>/<BRANCH>/
    ```

-   Link to the `LFS` repository content:

    ```text
    https://media.githubusercontent.com/media/<OWNER>/<NAME>/<BRANCH>/
    ```

-   Download release asset:

    ```text
    https://github.com/<OWNER>/<NAME>/releases/download/<TAG>/<FILE-NAME>
    ```

-   Download branch as archive:

    ```text
    https://github.com/<OWNER>/<NAME>/archive/<BRANCH>.tar.gz
    ```

-   Download tag as archive:

    ```text
    https://github.com/<OWNER>/<NAME>/archive/refs/tags/<TAG>.tar.gz
    ```

### Move repository

```shell
git remote rm origin
git remote add origin <URL>
git push origin --all
git push --tags
```

### Sync forked repository

```shell
git clone ssh://<YOUR-FORKED-REPO>
cd <YOUR-FORKED-REPO>
git remote add upstream https://<ORIGINAL-REPO>
git fetch upstream
git pull upstream main
git push

```

### Rename branch

In the following example we rename `master` to `main`:

```shell
git sw master
git branch -m main
git push origin -u main
git push origin --delete master
```

On `GitHub` repository `settings/branches` switch default branch to the `main`.

### Restore deleted file

```shell
FILE=<FILE-PATH>

git checkout $(git rev-list -n 1 HEAD -- "$FILE")^ -- "$FILE"
```

### Remove untracked files

```shell
# files
git clean --force

# directories
git clean --force -d
```

### Delete commited changesets

```shell
git reset --hard <sha1-commit-id>
```

If changesets were pushed:

```shell
git push origin HEAD --force
```

### Delete file from commits

```shell
git filter-repo --force --partial --invert-paths --path-match <FILE-PATH>

git reflog expire --expire-unreachable=now --all

git gc --prune=now --aggressive
```

### Chnod

Mode: XXXYYY

where:

-   XXX:
    100 - regular file
    120 - symlink

-   YYY:
    644 - rw-r--r--
    755 - rwxr-xr-x

```shell
# list execulable files
git ls-files --format "%(objectmode) %(path)" | grep "^100755"

# make file in "_tests" and "bin" directories executable
git ls-files _tests bin | xargs -d "\n" git update-index --chmod=+x

# make ".sh" file executable
git ls-files | grep ".sh$" | xargs -d "\n" git update-index --chmod=+x
```
