# awesome-aws-docs
One liner to download / spider / mirror a local copy of `docs.aws.amazon.com` from the Internet Archive, applications such as clustering / natural language processing / machine learning or just offline reading. Only downloads `/latest/`. Remove `--only \.html' if you also want images, json, etc.

This works in AWS CloudShell, ie, from your browser.

First install: https://github.com/hartator/wayback-machine-downloader

`wayback_machine_downloader --only '\.html' --only latest --exclude '/docs\.aws\.amazon\.com\/[a-z]+_[a-z]+\//' --concurrency 5 https://docs.aws.amazon.com`
