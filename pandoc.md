# Pandoc

[Pandoc](https://pandoc.org/demos.html) is a very likable tool to convert between many text formats like markup, word, html, jira, etc.

## Useful commands / tldr

```bash
# git bash windows: read jira markup from clipboard and return it as markdown to stdout
cat /dev/clipboard | pandoc -f jira -t markdown
# converting a webpage to markdown and output as stdout
pandoc -s -r html https://docs.oracle.com/en/database/oracle/oracle-database/19/vldbg/view-info-partition-tables-indexes.html#GUID-2D424638-511C-4CC3-9BDE-53FFB1686ECD -t markdown
```

## Comvert options

From all the [convert options](https://pandoc.org/chunkedhtml-demo/3.1-general-options.html), here are the [markdown variants](https://pandoc.org/MANUAL.html#markdown-variants):

- `markdown` (Pandoc’s Markdown)
- `markdown_mmd` (MultiMarkdown)
- `markdown_phpextra` (PHP Markdown Extra)
- `markdown_strict` (original unextended Markdown)
- `gfm`  (GitHub-Flavored Markdown)
- `commonmark` (CommonMark Markdown)
- `commonmark_x` (CommonMark Markdown with extensions)
