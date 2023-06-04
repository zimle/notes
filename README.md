# About

This is a simple repo to store personal notes about stuff I have touched and might need to remember. I like to save these notes as markdown files due to

- simplicity
- readability
- convertibility (e.g. via [pandoc](pandoc.md))
- greppability

Note that they are by way *not* written by an expert, so use with care!

## Set up

Please ensure that the [pre-commit hook](git-hooks/pre-commit) runs by configuring

```bash
git config core.hooksPath git-hooks
```

Note that currently [ripgrep](https://github.com/BurntSushi/ripgrep) is used instead of [grep](https://www.gnu.org/software/grep/), as grep does not seem to support `[:ascii:]` on the git bash. Simply aliasing `rg=grep` should also work on Unix.
