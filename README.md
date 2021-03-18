# awesome-aws-docs
> Difficulty: Advanced


One-liner to download / spider / mirror / crawl a local copy of `docs.aws.amazon.com` from the Internet Archive, for applications such as clustering / natural language processing / machine learning or just offline reading. Only downloads `/latest/` and only in english. Remove `html` from the command if you also want images, json, etc. The filtering reduces the number of pages downloaded from 1M+ to just ~85,000 by carefully eliminating garbage URIs, such as those that include `:80` or `?`. Note that this corpus will have the latest version of the document crawled by the Internet Archive, additionally, it may also have the last published page of documentation that is no longer active on `docs.aws.amazon.com`.

## Installation

```bash
#!/bin/bash
# For Ubuntu
sudo apt-get -y install ruby rdfind p7zip-full
sudo gem install wayback_machine_downloader
wget https://repo.anaconda.com/archive/Anaconda3-5.3.1-Linux-x86_64.sh -O ~/anaconda.sh
bash ~/anaconda.sh -b -p $HOME/anaconda
echo 'export PATH=$PATH:~/anaconda/bin' >> ~/.bashrc
source ~/.bashrc
conda update -n base -c defaults conda
```

## Spidering / Crawling / Mirroring

You should have the site by the same time the next day.

`time wayback_machine_downloader --only '/^https:\/\/docs\.aws\.amazon\.com\/.+\/latest\/.+\.html$/' --exclude '/docs\.aws\.amazon\.com\/[a-z]+_[a-z]+\//' --concurrency 5 https://docs.aws.amazon.com`

### Processing the corpus

You can remove any duplicate files (identical contents) with `rdfind`:

```
rdfind -deleteduplicates true websites
```

Remove any files or directories with '?' that made it in:

```
find . | grep '?' | xargs -L1 rm -fr
find . | grep '?' | xargs -d "\n" -I {} rm -fr '{}'
```

Check how many pages of documentation we ended up with, in this case, `86,570`:

```
find . | grep html | wc -l
```

Compute an estimate of the amount of information in the corpus (and create a compressed archive), in this case `677MB`:

```
7z a -t7z -m0=lzma -mx=9 -mfb=64 -md=32m -ms=off ~/docs.aws.amazon.com.7z ~/websites/docs.aws.amazon.com/
```



## Background

- `docs.aws.amazon.com` does not lend itself to spidering. 'View source' on the main page shows it is rendered with javascript, which would reqiure a headless spider. The ones that actually work cost money.
- They do publish sitemaps, but they are not exhaustive, and if you try to spider the docs directly, you will quickly be blocked.
- Some of the docs are on GitHub, but are spread across hundreds of repos.
- If you increase the concurrency above 5, you'll probably get blocked by the Internet Archive.
