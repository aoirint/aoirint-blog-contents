---
title: Sphinx
date: '2021-01-05T07:40:00+09:00'
updated: '2021-11-13T18:00:00+09:00'
draft: false
channel: 技術ノート
category: 'Get started'
tags:
  - 'Get started'
  - Python
  - Sphinx
  - Documentation
  - Blogging
---

# Sphinx

Python製のドキュメント生成ツール。


- Python 3.8.5
- Sphinx 3.4.2
  - [Sphinx · PyPI](https://pypi.org/project/Sphinx/)

```bash
pip3 install Sphinx
```


## reST（reStructuredText）

SphinxではデフォルトでreStructuredTextというマークアップ言語を使う。

- https://www.sphinx-doc.org/ja/master/usage/restructuredtext/basics.html
- https://atom.io/packages/language-restructuredtext


## 既存Pythonプロジェクトにドキュメントを追加する

`setup.py` などが存在するPyPIパッケージプロジェクトを想定する。

プロジェクトのルートに `docs` ディレクトリを作成し、
インタラクティブツール `sphinx-quickstart` を実行する。

```bash
mkdir docs
cd docs/
sphinx-quickstart
```

`Separate source and build directories (y/n)` と聞かれるので、 `y` 。
プロジェクト名、著者名、言語などを答える。

`source` ディレクトリ、 `build` ディレクトリ、 `Makefile` が生成される。

```bash
make html
```

以上のコマンドで `build/html` ディレクトリにHTMLが生成される。

```bash
python3 -m http.server -b localhost -d build/html
```

などで確認する。


次はPythonモジュールのdocstringからドキュメントを自動生成する。

- [Sphinx でPythonのAPIドキュメントを自動作成 - Qiita](https://qiita.com/some-nyan/items/1980198a05c12d90e5c3)
- [python書くなら絶対に使いたい2つのドキュメント生成ツール - Qiita](https://qiita.com/hatsumi3/items/11c5bc835efe713e4767)

```bash
#!/bin/bash

SCRIPT_DIR=$(cd $(dirname $0); pwd)

cd "${SCRIPT_DIR}"
sphinx-apidoc -f -o "./source/api" "../mymodule"

make html
```

`docs/mkdocs.sh` を以上のように作成し、実行する（TODO：Makefileへの入れ込み）。
ここでは、以下のように `docs` ディレクトリと並んで、パッケージとして提供するPythonモジュール `mymodule` のディレクトリがあることを想定している。
なお、 `docs/source/conf.py` は設定ファイルであり、
例えば `html_theme = 'sphinx_rtd_theme'` のような設定を追加し、
`pip3 install sphinx-rtd-theme` してから `make html` することで、Read the Docsスタイルのドキュメントを生成できる。

```
|- setup.py
|
|- docs/
  |- Makefile
  |- mkdocs.sh
  |
  |- source/
    |- conf.py
    |- index.rst
  |
  |- build/
    |- html/
      |- index.html
|
|- mymodule/
  |- __init__.py
  |- mymodule.py
```

`sphinx-apidoc` によりPythonモジュールのドキュメントが `source/api` に自動生成され、
見出しにあたるページが `source/api/modules.rst` に生成される。
このページへのリンクがどこにもない、という旨のエラーが表示されているはずなので、
インデックスページの `source/index.rst` にこのページへのリンクを追加する。

```reStructuredText
.. toctree::
  :maxdepth: 2
  :caption: Contents:

  api/modules
```


## CI,CDを整備する

自動生成されるHTMLをソースコードと同じブランチでGit管理したくはないので、
`docs/build` ディレクトリを `.gitignore` に追加し、
GitHub ActionsやGitLab CIを使ってドキュメントの生成、GitHub PagesやGitLab Pagesへの自動デプロイを整備する。


## Docker化・GitHub Actions整備の例

- https://github.com/aoirint/sphinx-docs-test
