---
canonical_url: ./
title: 'スニペット: Ubuntu, pyenv環境でtkinterを使う'
# og_image:
# twitter_card: summary_large_image
og_description: 'Ubuntu, pyenv環境でtkinterを使う'
date: '2020-10-04 03:30:00'
draft: false
category: スニペット
tags:
  - Ubuntu
  - pyenv
  - tkinter
---
# スニペット: Ubuntu, pyenv環境でtkinterを使う

```sh
sudo apt install tk-dev python-tk python3-tk
pyenv install 3.8.6
pyenv global 3.8.6

python3 -m tkinter
```
