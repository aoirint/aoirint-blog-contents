---
canonical_url: ./
title: Atom Text Editor Setup
date: '2021-05-14 01:00:00'
updated: '2021-06-27 17:00:00'
draft: falses
category: Editor
tags:
  - Atom
  - テキストエディタ
---

# Atom Text Editor Setup

GitHubが開発するElectron製高機能テキストエディタAtomをセットアップする。

## Installation
### macOS, Windows
- [https://atom.io/](https://atom.io/)

### Ubuntu
- [https://flight-manual.atom.io/getting-started/sections/installing-atom/#platform-linux](https://flight-manual.atom.io/getting-started/sections/installing-atom/#platform-linux)

```shell
wget -qO - https://packagecloud.io/AtomEditor/atom/gpgkey | sudo apt-key add -
sudo sh -c 'echo "deb [arch=amd64] https://packagecloud.io/AtomEditor/atom/any/ any main" > /etc/apt/sources.list.d/atom.list'
sudo apt update

sudo apt install atom
```



## Key Binding Resolver (Ctrl + .)
キーバインドが重複してどれが動いているかわからないときに便利。


## Packages
パッケージにより機能拡張ができる。

- [https://atom.io/packages/](https://atom.io/packages/)

GUIによる操作

- Install: Edit > Preferences > Install
- Config: Edit > Preferences > Packages

CLIによる一括インストール

```shell
# Export Package List
apm list -b -e -i --no-v > atom_packages.txt

# Import Package List and Install
apm install --packages-file atom_packages.txt
```

おすすめパッケージリスト

```
# atom_packages.txt

Sublime-Style-Column-Selection
atom-beautify
atom-ide-ui
atom-terminus
autocomplete-python
convert-to-utf8
document-outline
file-icons
ide-python
ide-typescript
language-docker
language-graphql
language-julia
language-latex
language-nginx
latex
markdown-preview-plus
pdf-view
project-manager
project-view
project-viewer
tree-view-git-status
language-haskell
ide-haskell
```

### Sublime-Style-Column-Selection
- [sublime-style-column-selection](https://atom.io/packages/sublime-style-column-selection)

Shift + 左ボタンドラッグで矩形選択ができる

### atom-beautify
- [atom-beautify](https://atom.io/packages/atom-beautify)

デフォルトで入っているフォーマッタ

### atom-ide-ui
- [atom-ide-ui](https://atom.io/packages/atom-ide-ui)

IDE機能のコア

### atom-terminus
- [atom-terminus](https://atom.io/packages/atom-terminus)
- fork of [atom-terminal](https://atom.io/packages/atom-terminal)

Ctrl + Shift + Tでターミナルウインドウを開く

### autocomplete-python
- [autocomplete-python](https://atom.io/packages/autocomplete-python)

Python補完用（要Jedi or Kite）

Python Executable Paths: `/home/USER/.pyenv/shims/python`

```shell
pip3 install jedi
```

### convert-to-utf8
- [convert-to-utf8](https://atom.io/packages/convert-to-utf8)

CP932やEUC-JPのテキストファイルをUTF-8に変換する

### document-outline
- [document-outline](https://atom.io/packages/document-outline)

文書のアウトライン表示

### file-icons
- [file-icons](https://atom.io/packages/file-icons)

ファイルアイコン表示

### ide-python
- [ide-python](https://atom.io/packages/ide-python)

Python IDE機能

Python Executable: `/home/USER/.pyenv/shims/python`

### ide-typescript
- [ide-typescript](https://atom.io/packages/ide-typescript)

TypeScript IDE機能

デフォルトで同梱されているTypeScriptが使われるが、バージョンが古いため設定を変える。
そのまま使うと、Reactで`Cannot use JSX unless the '--jsx' flag is provided`などのエラーが出たり、
TypeScriptの新しい記法が構文エラーとして表示されることがある。

```shell
npm set prefix ~/.node
npm install -g typescript
```

TypeScript server path: `/home/USER/.node/lib/node_modules/typescript/lib/tsserver.js`

### language-docker
- [language-docker](https://atom.io/packages/language-docker)

Docker シンタックスハイライト

### language-graphql
- [language-graphql](https://atom.io/packages/language-graphql)

GraphQL シンタックスハイライト

### language-julia
- [language-julia](https://atom.io/packages/language-julia)

Julia シンタックスハイライト

### language-latex
- [language-latex](https://atom.io/packages/language-latex)

LaTeX シンタックスハイライト

### language-nginx
- [language-nginx](https://atom.io/packages/language-nginx)

nginx シンタックスハイライト

### latex
- [latex](https://atom.io/packages/latex)

LaTeX ビルド

Dockerで動くようにする記事: [https://blog.aoirint.com/entry/2020/atom_latex_docker/](https://blog.aoirint.com/entry/2020/atom_latex_docker/)

### markdown-preview-plus
- [markdown-preview-plus](https://atom.io/packages/markdown-preview-plus)

Markdown プレビュー

### pdf-view
- [pdf-view](https://atom.io/packages/pdf-view)

PDF プレビュー

### project-view
- [project-view](https://atom.io/packages/project-view)

プロジェクトツリー（tree-view）にプロジェクトパスを表示

### project-viewer
- [project-viewer](https://atom.io/packages/project-viewer)

GUIで編集できるプロジェクト管理ツール

### tree-view-git-status
- [tree-view-git-status](https://atom.io/packages/tree-view-git-status)

tree-viewにGit status（ブランチなど）を表示
