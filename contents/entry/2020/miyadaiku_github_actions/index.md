---
canonical_url: ./
title: 静的サイトジェネレータMiyadaiku + GitHub Actions + GitHub Pagesでブログを作る
# og_image:
# twitter_card: summary_large_image
og_description: 静的サイトジェネレータMiyadaikuを使ってブログ環境を整備する
date: '2020-09-09 09:48:39'
draft: false
category: 記事
tags:
  - Miyadaiku
  - Static Website
  - GitHub
  - GitHub Actions
  - 'CI CD'
---
# 静的サイトジェネレータMiyadaiku + GitHub Actions + GitHub Pagesでブログを作る

## 概要
新しく静的サイトジェネレータでブログ環境を整備した。
細かい使い方には触れないが、構成を書いておく。


## 静的サイトジェネレータとCI/CD

### 静的サイトジェネレータ
静的サイトジェネレータというのはSphinx（Python製）とかJekyll（Ruby製、GitHub Pages標準らしい）とかHugo（Go製）みたいなやつで、
MarkdownだとかreStructuredTextだとかのファイル群からHTMLを生成するツール。

- [Sphinx](https://www.sphinx-doc.org/)
- [Jekyll](https://jekyllrb.com/)
- [Hugo](https://gohugo.io/)


### Miyadaiku

[Miyadaiku](https://miyadaiku.github.io/)はPython製の静的サイトジェネレータ。
Flaskで使うテンプレートエンジンのJinja2が使えることが特徴みたい。Jinja2はDjangoのテンプレートエンジンに似ている。
テンプレート上でどんな変数が使えるかはMiyadaikuの領域なので、ドキュメント（とサンプルテンプレート、ソースコード）を見ていくしかないかも。

- [github:miyadaiku/miyadaiku](https://github.com/miyadaiku/miyadaiku)
- [pypi:miyadaiku](https://pypi.org/project/miyadaiku/)


### GitHub Actions

GitHub上のリポジトリに対してpushやPull Requestがなされた時に事前に指定した処理を実行することのできるGitHubの機能。
Jenkinsなどに近そう。また、GitLabにも同様の機能があったはず。
GitHubのサーバで動く仮想環境上でDockerのような使い方でテストケースの実行やリリースファイルのビルド、サーバへのデプロイ、
つまりCI（Continuos Integration、テストやビルドの自動化）/CD（Continuous Delivery、デプロイの自動化）を設定できる。
Pull Requestなどへの自動ラベル付けやSlackへの通知なんかも設定することがあるのかな。
この設定はYAMLファイルとしてGitリポジトリ内に保存しますが、秘密鍵/トークンなどの情報を参照するための機能もあるみたい。
バージョン管理システム的な点ではGitは分散型なので文書自体の分散バージョン管理はできるが、
GitHubが落ちたら解消するまで（手元にリポジトリがあっても）GitHub ActionsによるCI/CDができないのが難点な気がする。

- [GitHub Status](https://www.githubstatus.com/)


## 考えていること

### レンダリング後のHTMLファイルの分離

Markdownを書いている時に見えるところに（レンダリング後の）HTMLファイルを置きたくない。
できれば何も考えずに（Markdownで書いた）メモファイルを置くのに使っていた適当なディレクトリの上でコマンドを実行したら
HTTPサーバを介して（オプションで指定したテーマなどで）いい感じにレンダリングしてくれるようなものがいい
（HTMLファイル自体にファイルシステムからアクセスできる必要はない）と思っていた。

このHTMLはMarkdownファイル群とレンダリングのオプションだけでいつでも生成可能なので、
レンダリング後のファイル自体が見えなければ、後からこのファイルを何かの間違いで編集してしまって正規性（再レンダリングしても差分がない状態）が崩れてしまうことがない。
それからMarkdownファイルをGitで管理する場合、レンダリング後のファイルの差分には実質的に意味がないので、（見える）commitに含めたくない気持ちがある。

この部分は適切に.gitignoreやCI/CDを設定すれば大抵の静的サイトジェネレータで実現可能だろうと思う。
今回はGitHub Actionsを使って、GitHub上の仮想環境にリポジトリから文書を読み込んでHTMLを自動生成し、文書と履歴を共有しない別ブランチ（gh-pages）に自動でcommitされるように設定する。

GitHub Actionsを動かすには`.github/workflows`以下にYAMLファイルを配置する。例えばこのような感じ。GitHub Pagesへのデプロイ（gh-pagesブランチの更新）には[github:peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)を使っている。

```
#!yaml
# deploy.yml
name: Deploy

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      - name: Setup Python
        uses: actions/setup-python@v2
        with:
          # Version range or exact version of a Python version to use, using SemVer's version range syntax.
          python-version: 3.x

      - name: Install dependencies
        run: pip3 install -r requirements.txt

      - name: Run miyadaiku-build
        run: miyadaiku-build --output public .

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

`name`、`run`のところをコピーして増やせばコマンドを増やすことができる。


### テーマと文書の分離

テーマのリポジトリと文書のリポジトリを分離したい。
これをすることで、
文書のリポジトリでは本来の目的であるメモやブログの文章に関するcommitだけ（レンダリングの設定ファイルは残ってしまうかもだが）、
テーマのリポジトリでは本来の目的であるHTMLテンプレートに関するcommitだけが
履歴として見えるようになる。
また、テーマのリポジトリだけを公開する、というようなことができるようになる（公開しなくてもgit経由でpip installしたりすればいい）。

今回はテーマとして[aoirint-miyadaiku-theme-blog](https://github.com/aoirint/aoirint-miyadaiku-theme-blog)を作成した。ついでにPyPIにも登録した。

- [pypi:aoirint-miyadaiku-theme-blog](https://pypi.org/project/aoirint-miyadaiku-theme-blog/)

### 文書の書きやすさ

日付の表示やタグ検索などレンダリング時の質を上げたければ、どうしても文書側にメタデータを付加しなければならないところがネックではある気はするが、
GitHub上でJekyll的なYAMLヘッダ（Miyadaikuも同様）のついたMarkdownを表示してみたところ、メタデータを整形表示してくれた。
この様子ならある程度互換性が期待できるし、そういう習慣をつけるのもいいのではないかなと思う。

- [Viewing YAML Metadata in your Documents - The GitHub Blog](https://github.blog/2013-09-27-viewing-yaml-metadata-in-your-documents/)

PC上で文書を書いているときには、ブラウザの更新がかかったり、ネットワークが不安定で再起動が必要になったり、OSの更新で勝手に再起動したり、寝落ちしてバッテリが切れたりなど、Webページが強制リロードしたり、エディタが強制終了したりすることがよくある。こういうときに書きかけの文書をできる限り保全してくれる仕組みがほしいところ。GitHub上で書いているとそこが若干不安。この点ではHackMDを使うのがよさそうに思う（コピーしたファイルを編集する形で、書きかけでもサービス上に自動保存してくれるみたい）が、CommitterがHackMDになってしまうのが気になるところ（同時編集するときはこれでよさそうだが）。それから、画像を外に置かないと扱いにくそう..。

- [Changing author info - GitHub Docs](https://docs.github.com/en/github/using-git/changing-author-info)

またUbuntu、Mac、WindowsのPCを時と場所と気分と目的によって切り替えて使っていて、タイミングによっては特定のPCにアクセスできないようなこともあるので、基本的に文書がリモートにないと面倒なのだが、（あんまり分散管理という感じがしないが）GitHubを経由して同期すればいいというのがGitを使う利点の一つであるように思う。

それから、複数の書きかけの文書/記事があるとき、何度も変更して大量に履歴ができたり、複数の記事の履歴が入り乱れるとつらそう。これは記事ごとにブランチを切る（あとでmergeする）とか、squash/rebaseを使うとか、Pull Requestを使うとかしてうまく対処できないか考えている。


#### 複数のコミットを1つにまとめる例

このような4つのコミット履歴のあるブランチがあるとする。

```
branch A: W--X--Y--Z
```

まず`git log --oneline`で対象のcommit IDを調べる。

```
ZZZZZZ COMMIT Z
YYYYYY COMMIT Y
XXXXXX COMMIT X
WWWWWW COMMIT W
```

`git rebase -i COMMIT_ID`は指定したIDのcommitの次のcommitから最新のcommitまでの履歴を操作するコマンド。エディタが開き、上から古い順にコミットが表示される。`git rebase -i WWWWWW`を実行すると、このようになる。

```
pick XXXXXX COMMIT X
pick YYYYYY COMMIT Y
pick ZZZZZZ COMMIT Z
```

頭の`pick`を`squash`か`s`に変えることで1つ上（前）のcommitに差分が統合される。保存してエディタを閉じるとコミットメッセージの編集が始まる。なお、失敗してエラーが出たときは`git rebase --abort`でrebaseを中断できる。

```
pick XXXXXX COMMIT X
pick YYYYYY COMMIT Y
s    ZZZZZZ COMMIT Z
```

```
branch A: W--X--Y'
```

すでにCOMMIT Y、COMMIT Zがリモートの同ブランチにpushされているときは`git push -f`する必要がある。

- [【git rebase -i】gitのcommitをまとめる - Qiita](https://qiita.com/tsuuuuu_san/items/f708a9f7ea8ab8eb6945)


#### 複数の記事を同時に書く際の例

まず、masterブランチ、記事Aを書いているブランチA、記事Bを書いているブランチBがあるとする。ブランチAとブランチBはmasterブランチの最終commit Xから分岐している。

```
master  : X
           \
branch A:   |--Y
branch B:   |--Z
```

ここで記事Aを書き終わったので、GitHub上でPull Requestを作成し、masterブランチにmergeした。ここでmasterブランチの最終commitはcommit Yになる。ブランチBは記事Bのcommit Zをすでに作成していて、masterブランチに対し、1 commit behind、1 commit aheadの状態になった。

```
master  : X--------Y
           \
branch B:   |--Z
```

ここでローカルでの作業に移りる。まずmasterブランチをcheckoutし、`git pull`してcommit履歴を最新にする。次にブランチBをcheckoutし、`git pull`してcommit履歴を最新にしたあと、`git rebase master`を実行することでcommit Yの次にcommit Zがくる1 commit aheadの状態にすることができる。ローカルでブランチBをいじっていた場合は、一度commitしてから同様にrebaseすれば同じことができる（`git rebase origin/master`というのもできる）。

```
master  : X--------Y
                    \
branch B:            |--Z
```

すでに古いcommit履歴のbranch Bがリモートにpushされているときは`git push -f`する必要がある。

-   [fast-forwardマージから理解するgit rebase - Qiita](https://qiita.com/vsanna/items/451b42f886c599a16a55)


## Miyadaikuを使う

```
#!bash
pip3 install miyadaiku
```

`miyadaiku==1.17.0`とする。
Miyadaikuをpipで入れると`miyadaiku-start`、`miyadaiku-build`コマンドが使えるようになる。

`miyadaiku-start`は空のMiyadaikuプロジェクトを作成するコマンド。といっても空の`files`、`templates`ディレクトリ、サンプルのMarkdownファイル`index.md`の入った`contents`ディレクトリ、デフォルトのシンプルなコンフィグファイル`config.yml`が生成されるだけ。

```
#!yaml
# Miyadaiku config file

# Base URL of the site
site_url: http://localhost:8888/

# Title of the site
site_title: FIXME - site title

# Default language code
lang: en-US

# Default charset
charset: utf-8

# Default timezone
timezone: Asia/Tokyo

# List of site theme
# themes:
#   - miyadaiku.themes.sample.blog
```

`files`ディレクトリは、汎用的な画像やCSS、JSを置くのに使う。このディレクトリの内容はそのまま出力ディレクトリにコピーされる。
`templates`ディレクトリには、Jinja2テンプレートを配置する。`contents`ディレクトリは、中に置いたMarkdownファイルなどがHTMLに変換されたものが出力ディレクトリに吐き出される。

`config.yml`の`themes`にpipで入れたテーマのモジュール名を書くことで、そのモジュール内の`contents`、`files`、`templates`が使えるようになる。`templates`内のHTMLテンプレート（`page_index.html`など）だけでなく、`contents`内の記事一覧を表示する`index.yml`などがテーマで用意されていれば、自分で作る必要はないみたい。

`miyadaiku-build`はHTMLファイルを生成するコマンド。デフォルトで`outputs`ディレクトリに吐き出される。
`--server`オプションで簡易サーバが立ち、`--watch`オプションで更新を自動検出する。`miyadaiku-build -sw .`で現在のディレクトリにある`config.yml`をもとに自動更新＆簡易サーバが立ち上がる。

`.gitignore`を次のようにして、記事用のリポジトリを作った。`_depends.pickle`というファイルはビルド時に生成されるが、バイナリなのでとりあえず載せないようにしている。

```
/_depends.pickle
/outputs
```

`config.yml`は`pip3 install aoirint_miyadaiku_theme_blog`したうえで、次のようにしている。

```
#!yaml
# Miyadaiku config file

# Base URL of the site
site_url: https://www.example.com

# Title of the site
site_title: SITE TITLE

# Default language code
lang: ja

# Default charset
charset: utf-8

# Default timezone
timezone: Asia/Tokyo

copyright: Copyright © 20xx HOLDER.

favicon: /favicon.ico

# ga_tracking_id: UA-*

twitter_name: example
github_name: github


# List of site theme
themes:
  - aoirint_miyadaiku_theme_blog

```

ディレクトリ構成はこのような感じ。

```
blog-repository/
|- .git/            # git管理
|- .github/         # GitHub Action用
|- contents/        # 記事を書くディレクトリ
   |- post1.md
|- files/           # 共通ファイルを置くディレクトリ
   |- favicon.ico
|- .gitignore
|- config.yml
|- requirements.txt # miyadaiku、テーマのインストール用
```

これで`miyadaiku-build -sw .`を実行すればデフォルトのポートに簡易サーバが立ち、実際のレンダリングをみることができる。
