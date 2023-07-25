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

- Typically, the global `.gitconfig` resides in the home folder. If one wants to make project specific configurations (like email address in the commit), one can still use the `--local` flag like this

    ```bash
    git config --local user.email my.special@address.com
    ```

    This adds to the local `.git/config` file of the repository. Note that the `--local` flag is the [default](https://git-scm.com/docs/git-config).

- Since [git 2.9 2016](https://stackoverflow.com/questions/14073053/find-path-to-git-hooks-directory-on-the-shell), one can config where the git hooks folder is (default `.git/hook`). Simply search in `git config --list --show-origin --show-scope` after the hook keyword

## Hooks

Hooks are a wonderful way to shift quality "to the left". There are [various hooks available](https://git-scm.com/docs/githooks), and on can see examples for them in the folder `.git/hooks`.
Removing the suffix `.sample` lets git execute these hooks.
E.g., by default, git looks for the file `.git/hooks/pre-commit` [before](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) one is forwarded to typing the commit message.

On the other hand, to enforce the quality checks for all team members, the pre-commit should be version controlled. To achieve this, one can create a `git-hooks` folder in the repository with all relevant hooks. To ensure that they are applied, run

```bash
git config core.hooksPath git-hooks
# on linux, the file must be executable, so don't forget
chmod +x git-hooks/pre-commit
```

Hint: When committing, one can [disable](https://git-scm.com/docs/git-commit) running the git commit hook via

```bash
# use --no-verify to prevent running the git commit hook
git commit --no-verify
```

Note that the hook will also not be applied if the file is not executable (like forgetting setting `chmod +x git-hooks/pre-commit` or alike).

Some of the standard frameworks for git hooks are

- [pre-commit](https://pre-commit.com/) (Python)
- [husky](https://typicode.github.io/husky/) (Javascript)

Those frameworks are also capable to provide checks and formatting that can be applied on other languages like Java.
On the other hand, one likes to stick with her / his toolchain (like using `gradle` (Java), `cargo` (Rust), `stack` (Haskell)).
Sometimes, one has to write the pre-commit hook himself.
Here is an example for Java:

```bash
#!/bin/sh

# apply gradle check for:
# - code formatting (spotless) on the files to be committed
# - code style (checkstyle)
# - code analysis (pmd)

undo_stashing() {
    echo "Pre-commit: Undo stashing"
    git stash pop --quiet
}

check() {
    ./gradlew checkstyleMain checkstyleTest pmdMain pmdTest --parallel
}
echo "Pre-commit hook: Stash working-tree and untracked files."
git stash save --quiet --include-untracked --keep-index

echo "Pre-commit hook: Format code changes"
if ! ./gradlew spotlessApply --parallel; then
    echo "Could not apply formatting. Please check the console output."
    undo_stashing
    exit 1
fi

# add changes by formatter to staging area
git add --all

echo "Pre-commit hook: Apply gradle checks"
if check; then
    echo "All checks passed."
    undo_stashing
    exit 0
else
    echo "A check failed. Please check the console output"
    undo_stashing
    exit 1
fi
```

The gist of such scripts is:

1. Clean all git areas except the ones to be committed (i.e. staging area aka index)
1. Apply formatter (if one wants to) and add changes to the staging area
1. Apply checks on the formatted code
1. Undo the stashing and return the untracked files as well as working tree changes

Only changes in the staging area and the formatter changes on that file (*whole* file in staging area will be formatted) will be committed.
Note, however, that after committing, your working tree might not have the changes from the formatter due to the stashing and only the formatted version is committed.
Thus, manual cleanup might still be needed.

To enforce those checks and not impose manual work for the developer, one can configure his toolchain to set up everything, e.g. with gradle:

```groovy
// build.gradle example
task setGitProjectConfig {
    // set git hook path to git-hooks, where git should look for its hooks like pre-commit
    exec {
        commandLine "git", "config", "--local", "core.hooksPath", "git-hooks"
    }
}

// every time java compiles (which a developer must do at some point), the git config is updated
compileJava.dependsOn processResources, setGitProjectConfig
```

## Stashing

Stashing is an easy method to temporarily remove changes from the working tree and/or the index (aka. staging area) and/or the untracked files.
Some easy use cases:

```bash
# temporarily remove changes from working tree and index
git stash
# see all temporarily removed changes
git stash list
# apply first stashed change back again and remove it from list
git stash pop
# apply 5-th stashed change from the list (git stash list) back again and remove it from list
git stash pop stash@{5}
# apply stash without removing it from the stash list
git stash apply stash@{5}
# stash with a meaningful name
git stash save name-i-can-remember
# apply stash with meaningful name
git stash apply name-i-can-remember
```

To play with different areas where stash will move the changes to "temp", consider the following scenario

```bash
git status
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   staged.java

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   unstaged.java

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        untracked.java
```

Behaviour with `git stash`:

```bash
# Stash diffs in staging area and working tree, new files will be untouched
git stash
git status
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        untracked.java

nothing added to commit but untracked files present (use "git add" to track)

# Note that the diffs from the staging area are now moved to working tree!
# Use git stash pop --index to bring back the changes from the index to the index
git stash pop
git status
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   staged.java
        modified:   unstaged.java

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        untracked.java

no changes added to commit (use "git add" and/or "git commit -a")
```

Behaviour with `git stash --keep-index`:

```bash
# Stash diffs only in working tree, new files and staged files will be untouched
git stash --keep-index
git status
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   staged.java

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        untracked.java

# Applying git stash pop will turn back to original setup
git stash pop
git status
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   staged.java

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   unstaged.java

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        untracked.java
```

Behaviour with `git stash --include-untracked`:

```bash
# Stash in all areas
git stash --include-untracked
git status
nothing to commit, working tree clean

# Applying git stash pop will turn back to original setup
git stash pop
git status
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   staged.java
        modified:   unstaged.java

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        untracked.java
```

Note that these commands can be combined:

```bash
# remove everything except in staging area
# useful for pre-commit hooks
git stash --keep-index -include-untracked
```

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
# show git history as graph / tree (adog)
git log --all --decorate --oneline --graph
```

## git diff

```bash
# diff a file (e.g. gradle.properties) between current workspace and branch
git diff other-branch -- build.gradle
# diff a file (e.g. gradle.properties) between branch-1 and branch-2
git diff branch-1 branch-2 -- build.gradle
# diff the last change of a file
git diff HEAD@{1} path/to/my/file.sql
```

## remote tracking

```bash
# find out which repo urls are tracked by local repo
git remote -v
# find out which local branches track which remote branches
git branch -vv
# set current branch to track on upstream branch foo
git branch -u upstream/foo
# set local branch bar to track on upstream branch foo
git branch -u upstream/foo bar
# Remove from local branch the link to the upstream branch that is tracked
git branch --unset-upstream
# Add repo on server (like github) as upstream to local branch
git remote add upstream https://github.com/jgm/pandoc.git
```

## Mass changes like formatting

When migrating to a formatter tool, the most simplest version is to just let it run over the whole code base.
Sadly, this annihilates a lot of useful information from git blame.
To circumvent this issue, one can create a list of commit hashes, typically called `.git-blame-ignore-revs`. One now has two options to ignore these commits in git blame:

1. Run `git config --local blame.ignoreRevsFile .git-blame-ignore-revs` such git automatically recognizes these commits.
    Even the IDE should recognize this.

2. Manually always use the flag `--ignore-revs-file` as in `git blame --ignore-revs-file .git-blame-ignore-revs`.

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

- Sometimes, files were deleted and created anew instead of using `git mv`, loosing references. For such cases, it is sometime more convenient to check out a specific commit and then undo this, e.g.:

    ```bash
    # get in detached state and sneak a consistent snapshot at the time of a specific commit
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

- Rename a file via `git mv MYOldfile.java MyOldFile.java`. This might become important for Windows, as it does not distinct between upper and lower case.

- Pull an upstream branch `my-branch` with changed history: Do not pull, as it automatically merges. Just use `git reset origin/main --hard`, which overrides your local history

- Show all files changed in the last 10 commits:

    ```bash
    # Show all changed files in the last 10 commits
    git diff --name-only --pretty="" HEAD~10 HEAD
    # Show all changed files in the last 10 commits, but exclude deleted files
    git diff --name-only --diff-filter=d --pretty="" HEAD~10 HEAD
    # Show only added files in the last 10 commits
    git diff --name-only --diff-filter=A --pretty="" HEAD~10 HEAD
    # Show only changed files in the last 10 commits, do not use a pager like less
    # useful for shell commands
    git --no-pager diff --name-only --pretty="" HEAD~10 HEAD
    ```

- See all authors and commits that have contributed to a file: `git shortlog -- MyFile.java`

- get the content of a file in the version of another branch: `git show other-branch:path/to/desired-file.md`
