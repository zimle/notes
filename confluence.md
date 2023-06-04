# Using the confluence API

## Useful sources

- [adding attachment via curl](https://confluence.atlassian.com/confkb/using-the-confluence-rest-api-to-upload-an-attachment-to-one-or-more-pages-1014274390.html)
- [Extracting an existing page id](https://confluence.atlassian.com/confkb/how-to-get-confluence-page-id-648380445.html)
- [confluence-rest-api-examples](https://developer.atlassian.com/server/confluence/confluence-rest-api-examples/)
- <https://developer.atlassian.com/cloud/confluence/rest/v1/api-group-content/>

## Code examples

Some nice commands (note that the base url `my_confluenc_base_url` is just the url you enter in the browser)

```bash
curl -u $user:$pwd "https://my_confluenc_base_url/rest/api/content/75860204/?expand=body.storage,version,space"
# Find out all spaces
curl -u $user:$pwd "https://my_confluenc_base_url/rest/api/space/"
```

## Tools

- [Mark](https://github.com/kovetskiy/mark) is a cli tool to publish markdown files to confluence
