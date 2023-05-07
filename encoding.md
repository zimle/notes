# Encoding problems

Encoding problems are pesky and can consume more time than one might think. Therefore, coding should always restrict to ASCII characters. The same holds generally to files that might be processed further, like these markdown files (e.g. if you want to convert them to word files or jira markup via [pandoc](pandoc.md)).

The following commands might prove useful:

```bash
# finding non-ascii characters, might not work on git bash
# grep can be replaced by ripgrep (https://github.com/BurntSushi/ripgrep)
grep --color='auto' -n "^[[:space:]]" my_file.md
# removing all non-ascii and control characters except for
# line feed \n and carriage return \r (for those Windows users)
cat my_file.md | tr -cd '[:print:]\r\n'
```
