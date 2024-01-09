# Shell / Bash / Zsh / Cli

## Functions

Functions in bash have no argument variables as in standard programming languages. Instead, one has to refer via the standard argument references:

- `$0` - Name of the script
- `$1` to `$9` - Arguments to the script. $1 is the first argument and so on.
- `$@` - All the arguments
- `${@:2}` - All the arguments except the first
- `$#` - Number of arguments
- `$?` - Return code of the previous command
- `$$` - Process identification number (PID) for the current script
- `!!` - Entire last command, including arguments. A common pattern is to execute a command only for it to fail due to missing permissions; you can quickly re-execute the command with sudo by doing sudo !!
- `$_` - Last argument from the last command. If you are in an interactive shell, you can also quickly get this value by typing Esc followed by . or Alt+.

Here is an example:

```bash
function join_by {
  local d=${1-} f=${2-}
  if shift 2; then
    printf %s "$f" "${@/#/$d}"
  fi
}
x=$(join_by '\n' "${@:2}")
echo "$x"
```

It also shows that functions do not have a return type. As more or less everything is a string, one has to catch the stdout of the function in a variable via `x=$(my_function)`.

## Escaping characters

The little [script](https://stackoverflow.com/questions/15783701/which-characters-need-to-be-escaped-when-using-bash) reveals all characters:

```bash
#!/bin/bash
special=$'`!@#$%^&*()-_+={}|[]\\;\':",.<>?/ '
for ((i=0; i < ${#special}; i++)); do
    char="${special:i:1}"
    printf -v q_char '%q' "$char"
    if [[ "$char" != "$q_char" ]]; then
        printf 'Yes - character %s needs to be escaped\n' "$char"
    else
        printf 'No - character %s does not need to be escaped\n' "$char"
    fi
done | sort
```

```bash
No, character % does not need to be escaped
No, character + does not need to be escaped
No, character - does not need to be escaped
No, character . does not need to be escaped
No, character / does not need to be escaped
No, character : does not need to be escaped
No, character = does not need to be escaped
No, character @ does not need to be escaped
No, character _ does not need to be escaped
Yes, character   needs to be escaped
Yes, character ! needs to be escaped
Yes, character " needs to be escaped
Yes, character # needs to be escaped
Yes, character $ needs to be escaped
Yes, character & needs to be escaped
Yes, character ' needs to be escaped
Yes, character ( needs to be escaped
Yes, character ) needs to be escaped
Yes, character * needs to be escaped
Yes, character , needs to be escaped
Yes, character ; needs to be escaped
Yes, character < needs to be escaped
Yes, character > needs to be escaped
Yes, character ? needs to be escaped
Yes, character [ needs to be escaped
Yes, character \ needs to be escaped
Yes, character ] needs to be escaped
Yes, character ^ needs to be escaped
Yes, character ` needs to be escaped
Yes, character { needs to be escaped
Yes, character | needs to be escaped
Yes, character } needs to be escaped
```

## Exit and return

From [SO](https://stackoverflow.com/questions/4419952/difference-between-return-and-exit-in-bash-functions):

> return will cause the current function to go out of scope, while exit will cause the script to end at the point where it is called. Here is a sample program to help explain this:

```bash
#!/bin/bash

retfunc() {
    echo "this is retfunc()"
    return 1
}

exitfunc() {
    echo "this is exitfunc()"
    exit 1
}

retfunc
echo "We are still here"
exitfunc
echo "We will never see this"
```

Note that return will give back the return value of the last called function `$?` if no return value is provided.

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

To profile changes, either use `zsh -xv` to enable `xtracing` and verbose `output` and look where the display gets hung (like loading `nvm`). To profile *time*, add  `zmodload zsh/zprof` at the beginning of your `.zshrc` file and `zprof` at the end.

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

Here are some other common use cases

```bash
# replace old with new for every case *in*-sensitive match 
sed 's/old/new/gI'
# replace comma between to digits by decimal point
sed -E 's/([[:digit:]]),([[:digit:]])/\1.\2/g'
```

## cdpath

Shells like the bash often provide a special path for changing directories: `$CDPATH`.
When doing `cd`, the shell will look for the first subdirectory match of the directories listed in `$CDPATH` and jump into this directory.

Lets imagine having the folder `open-source` somewhere:

```bash
ls /d/open-source/ | tr ' ' '\n'
assertj
junit5
localstack-java-utils
mark
modern-java-practices
pgjdbc
postgres
spring-batch
spring-integration
testcontainers-java
```

Now export `$CDPATH` in your `~/.bashrc`, `~/.zshrc` or whatever like `export CDPATH=$CDPATH:"/d/open-source"`.
After sourcing the rc file (like `source ~/.bashrc`), your shell will cd into specified directories automatically:

```bash
pwd
/c/Users/Me
cd postgres
pwd
/d/open-source/postgres
```

Note that even `export CDPATH=$CDPATH:".."` works, meaning that you always can jump into a "sibling" directory.

## Trivia

- find all classes-folder (windows) and using [rust version of find](https://github.com/sharkdp/fd):

    ```bash
    fd -pI 'classes' . | rg -o '.*\\classes' | sort | uniq
    ```

- Baeldung [link](https://www.baeldung.com/linux/job-control-disown-nohup) about running jobs in the background

- Some commands like `rm` do not offer a `--dry-run` flag, e.g. when using something destructive like `rm **/*.orig`. This can sometimes be imitated by using `echo **/*.orig`, as it is the shell expanding the globbing and not the command itself.

- download files

    ```bash
    # via curl
    curl -LO https://my-site.com/my-file.tar.gz
    # via wget
    wget https://my-site.com/my-file.tar.gz
    ```
