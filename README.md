# awesome-aws-docs
One liner to download / spider / mirror a local copy of `docs.aws.amazon.com` from the Internet Archive, for applications such as clustering / natural language processing / machine learning or just offline reading. Only downloads `/latest/` and only in english. Remove `--only \.html` if you also want images, json, etc.

This works in AWS CloudShell, ie, from your browser.

First install: https://github.com/hartator/wayback-machine-downloader

`wayback_machine_downloader --only '\.html' --only latest --exclude '/docs\.aws\.amazon\.com\/[a-z]+_[a-z]+\//' --concurrency 5 https://docs.aws.amazon.com`

## Background

- `docs.aws.amazon.com` does not lend itself to spidering. View source on the main page shows it is rendered with javascript, which would reqiure a headless spider. The ones that actually work cost money.
- They do publish sitemaps, but they are not exhaustive, and if you try to spider the docs directly, you will quickly be blocked.
- Some of the docs are on GitHub, but are spread across hundreds of repos.
