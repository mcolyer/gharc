# gharc

This is an experiment in trying to create a tool that would make it easy to use the stacked diff workflow on GitHub.

## Pre-requisites

- git 2.22
- jq
- hub

## User Story

Let’s say I have a local repository. Then I go and create my first diff:

```sh
$ git checkout master
$ git checkout -b newfeature
$ vim file.php
$ git commit # commit1
```

Now I want to request a review so I run:

```sh
gharc-diff
```

Which:

1. Looks for a git trailer indicating the review id, sees it doesn't have one, looks in `~/.gitarc` to determine the next id and adds one of `D1-newfeature`.
1. Saves the current `HEAD` to `$CURRENT_HEAD`
1. Runs `git branch gharc/D1-newfeature $CURRENT_HEAD`.
1. Runs `git push -u origin gharc/D1-newfeature:mcolyer/D1-newfeature`
1. Creates a pull request with a base of `master` and a head of `mcolyer/D1-newfeature`

To make a change to this diff, I:

```sh
$ vim file.php
$ git commit --amend
$ gharc-diff
```

Which:

1. Looks for a git trailer, finds `D1-newfeature`.
1. Saves the current `HEAD` to `$CURRENT_HEAD`
1. Runs `git checkout gharc/D1-newfeature`
1. Adds a new commit to `gharc/D1-newfeature` by applying `git diff $CURRENT_HEAD gharc/D1-newfeature`
1. Pushes `git push origin gharc/D1-newfeature:mcolyer/D1-newfeature`.


To start a new feature which depends on this feature, I stack the commits:

```sh
$ vim file.php
$ git commit # Fill out template
$ gharc-diff HEAD^
```

Which:

1. Saves the current `HEAD` to `$CURRENT_HEAD`
1. Looks for a git trailer indicating the review id, sees it doesn't have one, looks in `~/.gitarc` to determine the next id and adds one of `D2-newfeature`.
1. Runs `git branch gharc/D2-newfeature $CURRENT_HEAD`.
1. Runs `git push -u origin gharc/D2-newfeature:mcolyer/D2-newfeature`
3. Creates a pull request with a base of `mcolyer/D1-newfeature` and a head of `mcolyer/D2-newfeature`

In the meantime I've gotten feedback on my first diff and want to make changes:

```
git rebase -i master
git commit --amend
gharc-diff HEAD
git rebase --continue
```

When I call `gharc-diff` it:

1. Looks for a git trailer, finds `D1-newfeature`.
1. Saves the current `HEAD` to `$CURRENT_HEAD`
1. Runs `git checkout gharc/D1-newfeature`
1. Adds a new commit by applying `git diff gharc/D1-newfeature`
1. Pushes `git push origin gharc/D1-newfeature:mcolyer/D1-newfeature`.
1. Iterates over all `gharc/D*-newfeature`
    1. Rebases `gharc/DN-newfeature` on top of `gharc/D(N-1)-newfeature`.
    1. Pushes `git push origin gharc/DN-newfeature:mcolyer/DN-newfeature`.

TODO: After an approval on my first change, I land it by:

1. Runs `git checkout master`
1. Runs `git pull`
1. Runs `git checkout newfeature`
1. Runs `git rebase master`
1. Runs `git checkout master`
1. Runs `git merge --squash newfeature^`
1. Runs `git commit --reuse-message=newfeature^`

## Open Questions

1. Do people build on top of other's diffs?
1. Do people other than the author commit to diffs?
1. How do you merge things?