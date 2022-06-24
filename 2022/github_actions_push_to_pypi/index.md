---
title: GitHub ActionsでPyPIにPythonパッケージをpushする（GitHub Release連携でバージョン付け）
date: '2022-06-24 09:20:00'
draft: false
noindex: false
channel: 技術ノート
category: GitHub
tags:
  - GitHub
  - Python
---
# GitHub ActionsでPyPIにPythonパッケージをpushする（GitHub Release連携でバージョン付け）

- <https://qiita.com/aoirint/items/09ea153751a65bf4876f#github-release%E4%BD%9C%E6%88%90%E6%99%82%E3%81%ABpypi%E3%81%AB%E3%82%A2%E3%83%83%E3%83%97%E3%83%AD%E3%83%BC%E3%83%89>

上の記事のWorkflowテンプレートをちょっと改良した。

- GitHub Release作成時にリリースタグをパッケージバージョンにしてpush


## 構成

`mypackage/__init__.py`に以下のように開発用のバージョン情報を記述する。
リリース時にGithub Actionsでリリースタグに置換してからPyPIにpushする。

```python
__VERSION__ = '0.0.0'
```

## GitHub Secrets

- PYPI_API_TOKEN

## GitHub Workflow .github/workflows/pypi.yml

```yaml
name: Publish a package to PyPI

on:
  release:
    types:
      - created

env:
  VERSION: ${{ github.event.release.tag_name != '' && github.event.release.tag_name || '0.0.0' }}

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Setup Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.x

      - name: Install Dependencies
        run: |
            pip3 install -r requirements.txt
            pip3 install wheel
      - name: Replace version
        run: |
          sed -i "s/__VERSION__ = '0.0.0'/__VERSION__ = '${{ env.VERSION }}'/" mypackage/__init__.py
      - name: Build Package
        run: python3 setup.py sdist bdist_wheel

      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          user: __token__
          password: ${{ secrets.PYPI_API_TOKEN }}
```
