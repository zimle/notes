# Using the confluence API

## Useful sources

- [adding attachment via curl](https://confluence.atlassian.com/confkb/using-the-confluence-rest-api-to-upload-an-attachment-to-one-or-more-pages-1014274390.html)
- [Extracting an existing page id](https://confluence.atlassian.com/confkb/how-to-get-confluence-page-id-648380445.html)
- [confluence-rest-api-examples](https://developer.atlassian.com/server/confluence/confluence-rest-api-examples/)
- <https://developer.atlassian.com/cloud/confluence/rest/v1/api-group-content/>

## Code examples

Some nice commands (note that the base url `my_confluence_base_url` is just the url you enter in the browser)

```bash
curl -u $user:$pwd "https://my_confluenc_base_url/rest/api/content/75860204/?expand=body.storage,version,space"
# Find out all spaces
curl -u $user:$pwd "https://my_confluenc_base_url/rest/api/space/"
```

## Tools

- [Mark](https://github.com/kovetskiy/mark) is a cli tool to publish markdown files to confluence

## Bugs

- whenever a colon is contained in the page title, confluence [will not find attachments](https://jira.atlassian.com/browse/CONFSERVER-26803) (like in the multimedia macro)

- Confluence might not - depending on the browser - show videos that are [encoded with h265](https://confluence.atlassian.com/confkb/video-will-not-play-for-mp4-uploaded-files-1047539606.html). Audio will work. Instead, use the codec h264. Fair enough, this is actually no Confluence issue
