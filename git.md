# Git

## Partial clone

Also see

- <https://docs.gitlab.com/ee/topics/git/partial_clone.html>
- <https://git-scm.com/docs/partial-clone>
- <https://github.blog/2020-01-17-bring-your-monorepo-down-to-size-with-sparse-checkout/>
- <https://about.gitlab.com/blog/2020/03/13/partial-clone-for-massive-repositories/>

## Sparse checkout

## List of git configurations in current repo

To see all valid git configurations in the current, run

```bash
git config --list --show-origin --show-scope
```

which gives a list of all configs and where they come from (scope local, global, system and where the config files are).

## Maintenance / reducing git size

Git has now (since git 2.30.0) a dedicated [maintenance](https://git-scm.com/docs/git-maintenance) command which groups some usual existing clean-up tasks into one command. It also allows for scheduling etc. To run all tasks manually (as of version 2.40.0), execute

```bash
git maintenance run --task=commit-graph --task=gc --task=incremental-repack --task=loose-objects --task=pack-refs --task=prefetch
```

## Useful features

### Fix a commit that is only local

[To fix a specific commit](https://stackoverflow.com/questions/1186535/how-do-i-modify-a-specific-commit) that is *not* upstream (like correcting stuff that logically belongs to an older commit), do the following:

```bash
# find the commit you want to add stuff, e.g. e8adec4
git log .
# commit changes you want to merge into your commit, e.g. e8adec4
git commit -m "fixup! e8adec4"
# make an interactive rebase with --autosquash
git rebase e8adec4^ -i --autosquash
# the editor will open in correct order
pick e8adec4 Commit2
fixup 54e1a99 fixup! Commit2
pick b42d293 Commit3
```

## Caveats

- `git log .` might 'skip' commits. Simply use `git log` or - to check where a certain commit appears - run

    ```bash
    git branch -a --contains <commit-sha>
    ```
