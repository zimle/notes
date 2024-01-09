# ripgrep

[ripgrep](https://github.com/BurntSushi/ripgrep) is more or less a reimplementation of `grep` in `rust` which is faster and has more sensible defaults.

## Flags

```bash
# only print file (with path) where matches appear
rg -l foo
# case insensitive search
rg -i FoO
# do not print line number
rg -N foo
# also search in files ignored by git
rg --no-ingore foo
```
