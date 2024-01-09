# Pandoc

[Pandoc](https://pandoc.org/demos.html) is a very likable tool to convert between many text formats like markup, word, html, jira, etc.

## Useful commands / tldr

```bash
# git bash windows: read jira markup from clipboard and return it as markdown to stdout
cat /dev/clipboard | pandoc -f jira -t markdown
# converting a webpage to markdown and output as stdout
pandoc -s -r html https://docs.oracle.com/en/database/oracle/oracle-database/19/vldbg/view-info-partition-tables-indexes.html#GUID-2D424638-511C-4CC3-9BDE-53FFB1686ECD -t markdown
# convert markdown into javadoc (with small manual adaptions afterwards)
# read from stdin, write to stdout
pandoc -f markdown -t html | sed 's/^/ * /g'
```

Note that pandoc needs utf-8 as input and returns utf-8.
To store the output in clipboard with `clip` in windows, it is advisable to transform the output to `utf-16le` (UTF-16 with Little Endian BOM) and then store the result in clipboard like

```bash
cat /dev/clipboard | pandoc -f jira -t markdown | iconv -f utf-8 -t utf-16le | clip
```

## Convert options

From all the [convert options](https://pandoc.org/chunkedhtml-demo/3.1-general-options.html), here are the [markdown variants](https://pandoc.org/MANUAL.html#markdown-variants):

- `markdown` (Pandoc's Markdown)
- `markdown_mmd` (MultiMarkdown)
- `markdown_phpextra` (PHP Markdown Extra)
- `markdown_strict` (original unextended Markdown)
- `gfm`  (GitHub-Flavored Markdown)
- `commonmark` (CommonMark Markdown)
- `commonmark_x` (CommonMark Markdown with extensions)
