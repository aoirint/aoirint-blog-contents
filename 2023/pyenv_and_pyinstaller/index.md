---
title: 'pyenvでPyInstallerを使えるようにする'
date: '2023-04-16T11:03:47+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Python
tags:
  - pyenv
  - PyInstaller
---
# pyenvでPyInstallerを使えるようにする

```shell
env PYTHON_CONFIGURE_OPTS="--enable-shared" pyenv install 3.11.3
```

- [https://pyinstaller.org/en/stable/development/venv.html](https://pyinstaller.org/en/stable/development/venv.html)
