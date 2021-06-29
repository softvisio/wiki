# Git

### GitHub links

-   `https://<OWNER>.github.io/<NAME>/`

    Link to the docs site, returns data with the valid content type.

-   `https://raw.githubusercontent.com/<OWNER>/<NAME>/<BRANCH>/`

    Link to the content from the repository. Returns data **without** content type.

-   `https://media.githubusercontent.com/media/<OWNER>/<NAME>/<BRANCH>/`

    Link to the `LFS` repository content.

-   `https://github.com/<OWNER>/<NAME>/releases/download/<TAG>/<FILE-NAME>`

    Downlooad release asset

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
