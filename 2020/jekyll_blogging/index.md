---
# moved from https://aoirint.hatenablog.com/entry/2020/06/01/012251
title: Jekyll Blogging お試し
date: '2020-06-01T01:22:51+09:00'
draft: false
channel: 技術ノート
category: Jekyll
tags:
- Jekyll
- StaticWeb
---
# Jekyll Blogging お試し

- [https://jekyllrb.com/](https://jekyllrb.com/)

Ruby製の静的ウェブサイト生成ツール（Static Site Generator）。なんかMarkdownとかで書いたサイトをいい感じにHTMLにしてくれるやつ。

## Dockerイメージの準備

Ruby, RubyGems, gcc, makeが入っていれば動くらしい。公式Dockerイメージもあるみたいだけど、あえてスルーしてrubyイメージからやってみる。

ふだんRubyは使わないので試行錯誤。

- [https://hub.docker.com/_/ruby](https://hub.docker.com/_/ruby)

まずはイメージをビルド。

```dockerfile
FROM ruby:2

WORKDIR /code

RUN gem install jekyll bundler
```

ここで`gem install`してもいいのか、という問題がありそうだけどよくわからない..（キャッシュについてはいいとして）

```sh
sudo docker build . -t myjekyll
```

ひとまずこれでイメージの準備はできた。

```sh
sudo docker run --rm -v `pwd`/myblog:/code -e BUNDLE_PATH=vendor/bundle myjekyll jekyll new ./
```

これで`./myblog`（`/code`）に新しいJekyllプロジェクトが生成される（合わせて依存ライブラリが`myblog/vendor/bundle`にインストールされる）。

`vendor`を除いたフォルダ構成はこんな感じ。

```plain
myblog/
├── 404.html
├── about.markdown
├── _config.yml
├── Gemfile
├── Gemfile.lock
├── index.markdown
└── _posts
    └── 2020-05-31-welcome-to-jekyll.markdown
```

もし既存のプロジェクトを使う場合、`bundle install`でライブラリを取得する（Gemfileに書かれた依存ライブラリが`myblog/vendor/bundle`にインストールされる）。

```sh
sudo docker run --rm -v `pwd`/myblog:/code -e BUNDLE_PATH=vendor/bundle myjekyll bundle install
```

- [bundler、bundle execについて　※自分用メモ - Qiita](https://qiita.com/dawn_628/items/1821d4eef22b9f45eea8)
- [環境変数 BUNDLE_PATH の怪 - Qiita](https://qiita.com/ma2shita/items/700dd0bd229798f878b5)

最後に開発用サーバを立てる。

```sh
sudo docker run --rm -v `pwd`/myblog:/code -e BUNDLE_PATH=vendor/bundle -p 4000:4000 myjekyll bundle exec jekyll serve -H 0.0.0.0 -P 4000
```

`http://localhost:4000`でチェック。

## ディレクトリ構成

### ./

Gemfileなどがあるディレクトリ。ここはテンプレート（テーマ）置き場に使うみたい。

デフォルトで`index.markdown`ファイルはこうなっていた。

```md
---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---
```

### \_posts/

記事のMarkdownファイルをおく場所。

サンプル記事`2020-05-31-welcome-to-jekyll.markdown`の冒頭（適当に改行を入れた）はこうなっていた。

```md
---
layout: post
title:  "Welcome to Jekyll!"
date:   2020-05-31 13:16:10 +0000
categories: jekyll update
---
You’ll find this post in your `_posts` directory. Go ahead
and edit it and re-build the site to see your changes. You
can rebuild the site in many different ways, but the most
common way is to run `jekyll serve`, which launches a web
server and auto-regenerates your site when a file is updated.
```

ヘッダ部分（最初の`---`で囲まれた場所）を`Front Matter`というらしい（YAMLフォーマット）。ここにメタデータを書く。

### \_site/

ビルドされた静的サイトを構成するファイル群が出力される場所。

## 基本的なコマンド

### jekyll build

Jekyllが生成した静的サイトを構成するファイルは`_site`ディレクトリ以下に出力される。自分の作成したMarkdown記事からHTMLを生成するときは`jekyll build`を実行する。

### jekyll serve

開発用サーバを立ち上げるコマンド。生成された静的サイトを確認するときは`jekyll serve`を実行する。デバッガ（プレビュー）的な機能があるようで、変更が自動的に反映される（自動的に`jekyll build`してくれる）らしい。記事以外を編集した場合は反映されないみたい？

## 記事（Post）の作成

`_posts`ディレクトリ内に`.md`/`.markdown`ファイルを増やせばよい。サンプル記事に書いてあるのだが、ファイル名は`YEAR-MONTH-DAY-title.MARKUP`にする必要があるらしい。

`2020-05-31-my-first-post.md`

```md
---
layout: post
title:  "My First Post!"
date:   2020-05-31 22:05:00 +0900
tag:    First Post
tag:    Greeting
---
Hello Jekyll!

- [Jekyll](https://jekyllrb.com/)

```

## テーマの変更（例：Minimal Mistakes）

Jekyll 3.2からgemでデフォルトのテーマを導入するようになったらしい。デフォルトは`minima`。
そのため`_layouts`, `_includes`などのテーマ編集用のファイルが新規プロジェクトに生成されなくなった。

- [Directory Structure | Jekyll • Simple, blog-aware, static sites](https://jekyllrb.com/docs/structure/)

試しにテーマを`Minimal Mistakes`に変更してみる。

- [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/)

まずGemfileを編集し、以下の行を追加する。

```gem
# https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/
gem "minimal-mistakes-jekyll"
```

次に`_config.yml`の`theme`を変更するのだが、他にも色々とテーマに依存した設定をこのファイルに書くようなのでGitHubから持ってきて置き換えた方がいい気がする。

- [minimal-mistakes/_config.yml at master · mmistakes/minimal-mistakes](https://github.com/mmistakes/minimal-mistakes/blob/master/_config.yml)

```yaml
# theme: minima
theme: minimal-mistakes-jekyll
```

デフォルトの`index.markdown`をMinimal Mistakesの`index.html`に置き換える（`index.markdown`を削除、`index.html`をダウンロードして置き換えなど）

- [minimal-mistakes/index.html at master · mmistakes/minimal-mistakes](https://github.com/mmistakes/minimal-mistakes/blob/master/index.html)

投稿の`layout`が`post`から`single`になっているので各Markdownファイルを修正する（`about.markdown`も忘れずに、あるいは削除）。

それから、タグ検索ページを追加する。`_pages/tag-archive.md`を下からコピーしてくればOK（`_config.yml`はリポジトリのものに置き換えたと想定）。

- [minimal-mistakes/tag-archive.md at master · mmistakes/minimal-mistakes](https://github.com/mmistakes/minimal-mistakes/blob/master/docs/_pages/tag-archive.md)

これで`build`すればテーマが変わる。細かい作業が多くて結構面倒くさい..。Git管理することを考えて、できる限りもとのファイルを維持したまま置き換えようとしたからかな？　諦めて`_posts`だけを移行するようにして、丸ごと入れ替えてしまったほうが楽だったかもしれない。なんか昔のHTMLテンプレートを記事だけ使いまわせるようにした、みたいな...。

コマンドで新規記事をテンプレートから作成できそうなのも見かけたし、記事を書くときにも色々追加記法があって、それからテーマの作成についても書こうかと思っていたけど、また今度。
