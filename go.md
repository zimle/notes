# Go

## General

The [Go](https://go.dev/) language is a statically typed, compiled language designed at Google syntactically similar to `C`, but featuring garbage collection, structural typing and focus on concurrency.

## Usage without programming

Go has a very rich cli landscape and is very **easy to use** without knowing the eco system. To just use the code, the following suffices:

1. [Install](https://go.dev/doc/install) `Go` and add it to the path if not already done
1. Install the repo of your choice via `go install github.com/kovetskiy/mark@latest`
1. `Go` will create a directory `~/go` with the source dependencies in `~/go/pkg` and an executable of the repo in `~/go/bin`
1. If not already done, add `~/go/bin` to your path.
1. Check your executable, e.g. here `mark --version`

This resembles the way [pipx](https://pypa.github.io/pipx/) works for Python.
