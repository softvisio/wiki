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

### Move Repository

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

# in bitbucket repository settings change main branch to "main"

git push origin --delete master
```

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
