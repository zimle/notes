# Vim

## Often used key bindings

### Copy and paste multiple lines

1. press `SHIFT+v` resp. `V` to enter the visual line mode
1. select via `j` and `k` the lines to copy
1. press `y` (yank) or `x` (cut) to save to clipboard
1. escape and navigate to the place where to paste the lines
1. press `p` to paste all lines after the cursor or `P` to paste all lines before the cursor

### Duplicating a line

From [SO](https://stackoverflow.com/questions/73319/how-to-duplicate-a-whole-line-in-vim):

>Normal mode: see other answers.
>
>The Ex way:
>
> `:t.` will duplicate the line,
> `:t 7` will copy it after line 7,
> `:,+t0` will copy current and next line at the beginning of the file (,+ is a synonym for the range .,.+1),
> `:1,t$` will copy lines from beginning till cursor position to the end (1, is a synonym for the range 1,.).
>
>If you need to move instead of copying, use :m instead of :t.
>
>This can be really powerful if you combine it with :g or :v:
>
> `:v/foo/m$` will move all lines not matching the pattern "foo" to the end of the file.
> `:+,$g/^\s*class\s\+\i\+/t.` will copy all subsequent lines of the form class xxx right after the cursor.
>
>Reference: :help range, :help :t, :help :g, :help :m and :help :v

### Search and replace

Also see [Fandom](https://vim.fandom.com/wiki/Search_and_replace):

```bash
# Find each occurrence of 'foo' (in the current line only), and replace it with 'bar'.
:s/foo/bar/g
# Find each occurrence of 'foo' (in all lines), and replace it with 'bar'.
:%s/foo/bar/g
# Change only whole words exactly matching 'foo' to 'bar'; ask for confirmation.
:%s/foo/bar/g
```

## Using vimdiff

Vim can als be used to show and edit the diffs between two files via running `vim -d file1.txt file2.txt`. `vimdiff` is just a synonym for `vim -d`. It is also useful to solve merge conflicts. Here are some basics copied from [SO](https://stackoverflow.com/questions/14904644/how-do-i-use-vimdiff-to-resolve-a-git-merge-conflict):

>First, to address the "abort everything" option - if you do not want to use "vimdiff" and want to abort the merge: press `Esc`, then type `:qa!` and hit `Enter`. (see also How do I exit the Vim editor?). Git will ask you if the merge was complete, reply with n.
>
> If you want to use vimdiff, here are some useful shortcuts. This assumes you know basics of Vim (navigation and insert/normal mode):
>
> - navigate to the bottom buffer (merge result): `Ctrl-W` `j`
> - navigate to next diff with `j`/`k`; or, better, use `]` `c` and `[` `c` to navigate to the next and previous diff respectively
> - use `z` `o` while on a fold to open it, if you want to see more context
> - for each diff, as per @chepner's answer, you can either get the code from a local, remote or base version, or edit it and redo as you see fit
>   - to get it from the local version, use: `:diffget LO`
>   - from remote: `:diffget RE`
>   - from base `:diffget BA`
>   - or, if you want to edit code yourself, get a version from local/remote/base first, and then go to the insert mode and edit the rest
> - once done, save the merge result, and quit all windows :wqa
>         -if you want to abort merging the current file and not mark it as resolved, quit with `:cquit` instead: How do you cancel an external git diff?
> - normally, git detects that the merge was made and creates the merge commit

Note that the merge conflict schema is always the following:

```quote
    +--------------------------------+
    | LOCAL  |     BASE     | REMOTE |
    +--------------------------------+
    |             MERGED             |
    +--------------------------------+
```

Recap the meaning:

- local: the version before a git action like merge, rebase, stash pop was done
- remote: the version to apply to local like the branch to merge in, the head of the rebase one wants to get atop of, the stashed version
- base: the version that is common to both local and remote
- merged: the version you finally get when resolving the conflicts / you are quitting vimdiff

Additional key bindings:

- use `CTRL+w` generally to switch between windows
