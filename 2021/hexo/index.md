---
title: Hexo
date: '2021-11-17 04:00:00'
draft: false
category: Blogging
tags:
  - Blogging
---
# Hexo

- <https://hexo.io/docs/#Installation>

```shell
npm install -g hexo-cli
```

```shell
$ hexo help
Usage: hexo <command>

Commands:
  help     Get help on a command.
  init     Create a new Hexo folder.
  version  Display version information.

Global Options:
  --config  Specify config file instead of using _config.yml
  --cwd     Specify the CWD
  --debug   Display all verbose messages in the terminal
  --draft   Display draft posts
  --safe    Disable all plugins and scripts
  --silent  Hide output on console

For more help, you can use 'hexo help [command]' for the detailed information
or you can check the docs: http://hexo.io/docs/
```

`hexo init <folder>`は空のディレクトリに対して実行する必要がある。
いったん仮のディレクトリを作ってからコピーする。`.github`、`.gitignore`ファイルがあるのでコピー忘れに注意。

- <https://hexo.io/docs/setup>

```shell
$ hexo init .
FATAL ~/git/vent.aoirint.com not empty, please run `hexo init` on an empty folder and then copy your files into it
FATAL {
  err: Error: target not empty
...
```

ローカルサーバーを起動する。

```shell
hexo server
```

テーマをlightに変更する。

- <https://github.com/hexojs/hexo-theme-light>

```shell
git clone --depth 1 https://github.com/hexojs/hexo-theme-light themes/light
```

```yaml
# _config.yml
theme: light
```

静的ファイルを生成する。

```shell
hexo generate
```

GitHub上のgh-pagesブランチにデプロイする設定をする。

- <https://hexo.io/docs/one-command-deployment.html>

```shell
npm install hexo-deployer-git
```

```yaml
deploy:
  type: git
  repo: https://github.com/myuser/myrepo
  branch: gh-pages
```

生成した静的ファイルをデプロイする。

```shell
hexo deploy
```


記事を限定公開化するhiddenオプションをfront-matterに追加する。

- <https://prokou.caitsith.info/hexo/setup.html#%E8%A8%98%E4%BA%8B%E3%82%92%E9%9A%A0%E3%81%99>
- <https://github.com/prinsss/hexo-hide-posts>

```shell
npm install hexo-hide-posts
```

外部リンクにnoreferrerを追加する。

- <https://github.com/hexojs/hexo-filter-nofollow>

```
npm install hexo-filter-nofollow
```
