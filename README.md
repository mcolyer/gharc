# gharc

This is an experiment in trying to create a tool that would make it easy to use the stacked diff workflow on GitHub.

## User Story

Let’s say I have a local repository. Then I go and create my first diff:

```sh
touch app/models/car.rb
git commit -am “Add Car model” #commit1
```

Now I want to request a review so I run:

```sh
gharc diff
```

Which:

1. Creates a branch called `mcolyer/add-car-model`.
2. Set HEAD of `mcolyer/add-car-model` to sha of `commit1`.
3. Creates a pull request with a base of `master` and a head of `mcolyer/add-car-model`

Now I keep working and want to add a new type of car, a sports car:

```sh
touch app/models/sports_car.rb
git commit -am “Add SportsCar model” #commit2
```

Now I'm ready for review:

```sh
gharc diff
```

Which:

1. Creates a branch called `mcolyer/add-sportscar-model`.
2. Set HEAD of `mcolyer/add-sportscar-model` to sha of `commit2`
3. Creates a pull request with a base of `mcolyer/add-car-model` and a head of `mcolyer/add-sportscar-model`

In the meantime I've gotten feedback on my first review and want to make changes:

```
git rebase -i (select first commit)
vim app/model/car.rb
git add app/model/car.rb
git rebase --continue
```

Now I'm ready for review of that change so I run `gharc diff` which:

1. Adds a new commit to `mcolyer/add-car-model` by applying `git diff <sha-of-add-car-model-on-master> <sha-of-add-car-model-on-branch>`
1. Rebases `mcolyer/add-sportscar-model` on top of `mcolyer/add-car-model`.
1. Pushes `mcolyer/add-car-model`.
1. Pushes `mcolyer/add-sportscar-model`.

After an approval on my first change, then I run `gharc land 1` (where the PR is labelled one), which:

1. Runs `git checkout mcolyer/add-car-model`
1. Runs `git pull` (maybe not necessary?)
1. Runs `git rebase master`
1. Runs `git checkout master`
1. Runs `git merge --squash mcolyer/add-car-model`
1. Runs `git commit --reuse-message=<commit1-sha>`

## Open Questions

1. Do people build on top of other's diffs?
1. Do people other than the author commit to diffs?