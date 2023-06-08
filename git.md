# Git

## Partial clone

Also see

- <https://docs.gitlab.com/ee/topics/git/partial_clone.html>
- <https://git-scm.com/docs/partial-clone>
- <https://github.blog/2020-01-17-bring-your-monorepo-down-to-size-with-sparse-checkout/>
- <https://about.gitlab.com/blog/2020/03/13/partial-clone-for-massive-repositories/>

## Sparse checkout

## List of git configurations in current repo

To see all valid git configurations in the current repository, run

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

- To see every configuration set, its scope and where it is located, use

    ```bash
    git config --list --show-origin --show-scope
    ```

- To unset a specified option (e.g. hookspath), use

    ```bash
    git config --local --unset core.hookspath
    ```

- Typically, the global `.gitconfig` resides in the home folder. If one wants to make project specific configuartions (like email address in the commit), one can still use the `--local` flag like this

    ```bash
    git config --local user.email my.special@address.com
    ```

    This adds to the local `.git/config` file of the repository. Note that the `--local` flag is the [default](https://git-scm.com/docs/git-config).

- Since [git 2.9 2016](https://stackoverflow.com/questions/14073053/find-path-to-git-hooks-directory-on-the-shell), one can config where the git hooks folder is (default `.git/hook`). Simply search in `git config --list --show-origin --show-scope` after the hook keyword

## Hooks

Hooks are a wonderful way to shift quality "to the left". There are various hooks availbale, and on can see examples for them in the folder `.git/hooks`. Removing the suffix `.sample` lets git execute these hooks. E.g., by default, git looks for the file `.git/hooks/pre-commit` [before](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) one is forwarded to typing the commit message.

On the other hand, to enforce the quality checks for all team members, the pre-commit should be version controlled. To achieve this, one can create a `git-hooks` folder in the repository with all relevant hooks. To ensure that they are applied, run

```bash
git config core.hooksPath git-hooks
# on linux, the file must be executable, so dont forget
chmod +x git-hooks/pre-commit
```

Hint: When committing, one can [disable](https://git-scm.com/docs/git-commit) running the git commit hook via

```bash
# use --no-verify to prevent running the git commit hook
git commit --no-verify
```

Note that the hook will also not be applied if the file is not executable (like forgetting setting `chmod +x git-hooks/precommit` or alike).

## git log

From [stackoverflow](https://stackoverflow.com/questions/4468361/search-all-of-git-history-for-a-string):

```bash
# find any commit that added or removed the string password, see also git log docs for 'pickaxe'
git log -S password
# looks for differences whose added or removed line matches the given regular expression,
# see also git log docs for 'pickaxe'
git log -G password
# to additionally see the diffs, use the flag -p (for patch), e.g.
git log -pG password
```

## git diff

```bash
# diff a file (e.g. gradle.properties) between current workspace and branch
git diff other-branch -- build.gradle
# diff a file (e.g. gradle.properties) between branch-1 and branch-2
git diff branch-1 branch-2 -- build.gradle
```

## remote tracking

```bash
# find out which local branches track which remote branches
git branch -vv
# set current branch to track on upstream branch foo
git branch -u upstream/foo
# set local branch bar to track on upstream branch foo
git branch -u upstream/foo bar
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

### Fix the commit message of an older commit not pushed yet

From [github](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/changing-a-commit-message):

```bash
# rebase up to the last commit you want to edit, e.g. 3
git rebase -i HEAD~3
# for each commit you want to change the message for, replace 'pick' with 'reword'
# change the messages
# congrats, you are done
```

### Rebase general

```bash
# rebase last two commits of current branch onto branch target_branch
git rebase -i HEAD~2 --onto target_branch
```

### Pushing

```bash
# push current HEAD to the branch upstream_branch on the upstream server origin
# This formulation might be necessary when the local branch has an other name than
# the upstream
git push origin HEAD:upstream_branch
```

### Fix the author / email address

If the email address in your last commit is not correct or wrong or not recognized by your upstream server (like checking it with your Active Directory email address that is an old one), simply change the settings in your `~/.gitconfig` (or according to `git config --list --show-origin --show-scope`) and run

```bash
git commit --amend --reset-author --no-edit
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

- Rename a branch via

    ```bash
    # rename current branch to new_name
    git branch -m new_name
    # rename branch named old_name to new_name
    git branch -m old_name new_name
    ```

- Pull an upstream branch `my-branch` with changed history: Do not pull, as it automatically merges. Just use `git reset origin/main --hard`, which overrides your local history
