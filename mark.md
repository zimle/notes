# Mark

## General

[Mark](https://github.com/kovetskiy/mark) is a cli tool to create html out of markdown files and publish them to Confluence. Mark supports certain "html tags" for meta data like

```html
<!-- Space: <space key> -->
<!-- Parent: <parent 1> -->
<!-- Parent: <parent 2> -->
<!-- Title: <title> -->
<!-- Attachment: <local path> -->
<!-- Label: <label 1> -->
<!-- Label: <label 2> -->
```

or inserting some macros like table of contents

```html
<!-- Include: ac:toc -->
```

## Mermaid

`Mark` also supports rendering [mermaid diagrams](https://mermaid.js.org/syntax/flowchart.html) in two ways

1. The Confluence plugin [Cloudscript.io Mermaid Addon](https://marketplace.atlassian.com/apps/1219878/cloudscript-io-mermaid-addon?tab=overview&hosting=cloud) is enabled. Then `mark` will use this plugin (default, to be explicit: `mermaid-go`)
1. Mark renders the mermaid diagrams itself, creates temporary images and pushes them to confluence. Use the flag `--mermaid-provider mermaid-go` for this

## Publishing to Confluence

[Mark](https://github.com/kovetskiy/mark) provides the following ways of running it:

- compile the project with `go`
- prebuilt binaries from the [releases](https://github.com/kovetskiy/mark/releases/) (only mac and linux)
- docker (`kovetskiy/mark:latest`)

I run it via go (it is just a `go install github.com/kovetskiy/mark@latest` and adding paths) and

```bash
# Hosted version
mark --mermaid-provider mermaid-go \
    --title-from-h1 \
    --drop-h1 \
    --minor-edit \
    --space "my-space" \
    --target-url https://my.base.confluence.url.com/ \
    -u user \
    -p pw \
    --dry-run \ # remove for actually pushing
    -f my_folder/*.md

# Cloud version
mark --mermaid-provider mermaid-go \
    --title-from-h1 \
    --drop-h1 \
    --minor-edit \
    --space "my-space" \
    --base-url https://my-company.atlassian.net/wiki/ \ # wiki seems to be important
    -u user \
    -p token \ # token is important, create in UI via Manage Account -> Security
    --dry-run \ # remove for actually pushing
    -f my_folder/*.md
```

Note that `mark` also provides the flag `--dry-run` to see the html output that would be generated, without actually creating the content.

## Personal

I personally do not aim primary for synching my notes (projects may!) with confluence and regard this only as a service. Therefore, I

- aim to reduce the necessary "html tags" by using cli flags as much as possible (e.g. `--title-from-h1` replaces `<!-- Title: <title> -->`)
- transform the original data into separate folder with necessary "html tags"

Here is my somewhat ugly script that does the job (must be started from some specific directory...):

```bash
#! /usr/bin/env bash

# copies all markdown files into a new folder, prepending
# html headers specific to https://github.com/kovetskiy/mark

set -euo pipefail
IFS=$'\n\t'

# check if we are in the parent of the notes and
# work-notes directory
# WARNING: Relative paths...
[[ -d ./notes ]] || { echo 'notes not found' ; exit 1; }
[[ -d ./work-notes ]] || { echo 'work-notes not found' ; exit 1; }

# create new folders, as we will copy the original md files
# into them with a mark specific header
notes_target_folder="./notes-confluence"
work_notes_target_folder="./work-notes-confluence"
mkdir -p "$notes_target_folder"
mkdir -p "$work_notes_target_folder"

# stolen from https://stackoverflow.com/questions/1527049/how-can-i-join-elements-of-a-bash-array-into-a-delimited-string
function join_by {
  local d=${1-} f=${2-}
  if shift 2; then
    printf %s "$f" "${@/#/$d}"
  fi
}

# appends specified arguments to the top of the provided file
# and writes it back to the provided folder
# Arguments:
# 1. The file name
# 2. The folder to write in
# 3. - 9.: the headers to add to the file
add_to_header_and_write() {
    base_name=$(basename "$1")
    headers=$(join_by '\n' "${@:3}")
    sed "1s/^/$headers\n/" "$1" > "${2}/$base_name"
}

# export the function, as it is not accepted by xargs and we have to circumvent it
export -f add_to_header_and_write

# WARNING: mark seems to need a new line between html tags and plain md text, otherwise the line is dropped
disclaimer="\n**NOTE**: This document is generated, do not edit manually. Instead, shoot me a message for feedback!\n\n**WARNING**: These are personal notes taken because I am *not* an expert, so be careful!\n"

# transform files from different folders and save
# CAUTION: Separate disclaimer from other html tags by new line
find ./notes -name "*.md" -print0 | 
    while IFS= read -r -d '' line; do 
        echo "Processing $line"
        add_to_header_and_write "$line" \
                "$notes_target_folder" \
                '<!-- Parent: notes -->' \
                "$disclaimer" \
                '<!-- Include: ac:toc -->'
    done

find ./work-notes -name "*.md" -print0 |
    while IFS= read -r -d '' line; do 
        echo "Processing $line"
        add_to_header_and_write "$line" \
                "$work_notes_target_folder" \
                '<!-- Parent: work-notes -->' \
                "$disclaimer" \
                '<!-- Include: ac:toc -->'
    done

# publish documents
mark --mermaid-provider mermaid-go\
    --title-from-h1 \
    --drop-h1 \
    --minor-edit \
    --space "my-space" \
    --target-url https://my.base.confluence.url.com/ \
    -u user \
    -p password \
    -f "$notes_target_folder/*.md"

mark --mermaid-provider mermaid-go \
    --title-from-h1 \
    --drop-h1 \
    --minor-edit \
    --space "my-space" \
    --target-url https://my.base.confluence.url.com/ \
    -u user \
    -p password \
    -f "$work_notes_target_folder/*.md"

# clean up
echo "Deleting $notes_target_folder"
rm -rf "$notes_target_folder"
echo "Deleting $work_notes_target_folder"
rm -rf "$work_notes_target_folder"

```
