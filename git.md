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

## Configuration

Typically, the global `.gitconfig` resides in the home folder. If one wants to make project specific configuartions (like email address in the commit), one can still use the `--local` flag like this

```bash
git config --local user.email my.special@address.com
```

This adds to the local `.git/config` file of the repository.

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

### Fix the commit message of an older commit not pushed yet

From [github](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/changing-a-commit-message):

```bash
# rebase up to the last commit you want to edit, e.g. 3
git rebase -i HEAD~3
# for each commit you want to change the message for, replace 'pick' with 'reword'
# change the messages
# congrats, you are done
```

## Caveats

- `git log .` might 'skip' commits. Simply use `git log` or - to check where a certain commit appears - run

    ```bash
    git branch -a --contains <commit-sha>
    ```

## Trivia

- Sometimes, files were deleted and created anew instead of using `git mv`, loosing references. For such cases, it is sometime more convenient to check out a specifc commit and then undo this, e.g.:

    ```bash
    # get in detached state and sneak a consistent snapshot at the time of a specifc commit
    git checkout rev_id_i_want_to_see
    # undo the detached state and get back to the pointer of your current branch
    git switch -
    ```
