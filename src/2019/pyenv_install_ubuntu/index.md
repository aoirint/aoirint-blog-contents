---
# moved from https://aoirint.hatenablog.com/entry/2019/10/10/123924
title: ■
date: '2019-10-10T12:38:07+09:00'
draft: true
channel: 技術ノート
---
# ■

```shell
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
sudo apt install zlib1g-dev libssl-dev libffi-dev libbz2-dev libreadline-dev libsqlite3-dev
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' > ~/.bashrc
source ~/.bashrc
pyenv install 3.7.4
pyenv global 3.7.4
```
