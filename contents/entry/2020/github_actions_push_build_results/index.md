---
canonical_url: ./
title: 'スニペット: GitHub Actions, ビルド結果を別ブランチにpushする'
# og_image:
# twitter_card: summary_large_image
og_description: 'GitHub Actions実行中に生成した`build`ディレクトリの内容を別ブランチにpushする'
date: '2020-09-27 16:30:00'
draft: false
category: スニペット
tags:
  - GitHub
  - GitHub Actions
  - 'CI CD'
---
# スニペット: GitHub Actions, ビルド結果を別ブランチにpushする
GitHub Actions実行中に生成した`build`ディレクトリの内容を別ブランチにpushする。

- [github:s0/git-publish-subdir-action](https://github.com/s0/git-publish-subdir-action)

```
#!yaml
name: Build
on:
  push:
    branches:
      - master

jobs:
  deploy:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    - name: Setup Python
      uses: actions/setup-python@v2
      with:
        # Version range or exact version of a Python version to use, using SemVer's version range syntax.
        python-version: 3.x

    - name: Install dependencies
      run: pip3 install -r requirements.txt

    - name: Build
      run: python3 app/main.py
      # run: |
      #   python3 app/cmd1.py
      #   python3 app/cmd2.py

    # https://github.com/s0/git-publish-subdir-action
    - name: Push to build branch
      uses: s0/git-publish-subdir-action@master
      env:
        REPO: self
        BRANCH: build
        FOLDER: build
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
