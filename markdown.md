# Markdown

Markdown is an easy markup language to write structured text as plain text, which is rendered to html.
It is supported by gitlab and github.

## Emails

There seems to be no built-in way in pandoc to convert markdown to email html.
As a workaround, the following workflow stolen from the [obsidian forum](https://forum.obsidian.md/t/workflow-for-writing-emails-in-obsidian-and-sending-via-outlook-maybe-with-pandoc/43354) can suffice on **Windows**:

1. Copy/Paste your markdown to notepad++
1. Use the notepad++ plugin [MarkdownViewer++](https://github.com/nea/MarkdownViewerPlusPlus) to render the markdown (press the button MarkdownViewer++)
1. Click the button `Send/HTML to clipboard`
1. Paste into your email
