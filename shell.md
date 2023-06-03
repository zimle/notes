# Shell / Bash / Zsh / Cli

## shell is slow

Helpful links are

- [speeding up zsh](https://blog.jonlu.ca/posts/speeding-up-zsh)

To time it, run something like

```bash
/usr/bin/time zsh -i -c exit #If you want to time yours
```

Note that nvm incredibly slows the startup time down. Use the following config:

```bash
# nvm node version management
export NVM_DIR="$HOME/.nvm"
# loading nvm is incredibly slow
#[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
# only load it when needed
alias nvm="unalias nvm; [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"; nvm $@"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

To profile changes, either use `zsh -xv` to enable `xtracing` and verbose `output` and look where the display gets hung (like loading `nvm`). To profile *time*, add  `zmodload zsh/zprof` at the beginnig of your `.zshrc` file and `zprof` at the end.

## awk

```bash
# return from stream each line where column 1 does not match column 2, separation character being ;
awk -F ";" '$1 = $2'
```

## sed

A list of literal sets from [stackexchange](https://unix.stackexchange.com/questions/102008/how-do-i-trim-leading-and-trailing-whitespace-from-each-line-of-some-output):

```quote
[[:alnum:]]  - [A-Za-z0-9]     Alphanumeric characters
[[:alpha:]]  - [A-Za-z]        Alphabetic characters
[[:blank:]]  - [ \t]           Space or tab characters only
[[:cntrl:]]  - [\x00-\x1F\x7F] Control characters
[[:digit:]]  - [0-9]           Numeric characters
[[:graph:]]  - [!-~]           Printable and visible characters
[[:lower:]]  - [a-z]           Lower-case alphabetic characters
[[:print:]]  - [ -~]           Printable (non-Control) characters
[[:punct:]]  - [!-/:-@[-`{-~]  Punctuation characters
[[:space:]]  - [ \t\v\f\n\r]   All whitespace chars
[[:upper:]]  - [A-Z]           Upper-case alphabetic characters
[[:xdigit:]] - [0-9a-fA-F]     Hexadecimal digit characters
```

For example, to remove leading and trailing whitespace, use

```bash
sed 's/^[[:blank:]]*//;s/[[:blank:]]*$//'
```

## Trivia

- find all classes-folder (windows) and using [rust version of find](https://github.com/sharkdp/fd):

    ```bash
    fd -pI 'classes' . | rg -o '.*\\classes' | sort | uniq
    ```

- Baeldung [link](https://www.baeldung.com/linux/job-control-disown-nohup) about running jobs in the background
