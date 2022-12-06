---
title: 'npm startでError: ENOENT: no such file or directory ... .steampathで起動しないときの対処（React）'
date: '2021-06-05T01:30:00+09:00'
updated: '2021-08-22T22:30:00+09:00'
draft: false
channel: 技術ノート
category: React
tags:
  - Node.js
  - npm
  - React
---

# npm startでError: ENOENT: no such file or directory ... .steampathで起動しないときの対処（React）

gitでブランチを切り替えて依存関係に差分が生じたとき、
node_modulesを修正していないときに起きる。

```
Error: ENOENT: no such file or directory, stat '/home/USER/.steampath'
```

package-lock.jsonがあれば、
package-lock.jsonに基づいて依存関係をインストールする`npm ci`を実行する。

実行時に`node_modules`ディレクトリの中身は削除され、再構築される。

```shell
npm ci
```

- [https://docs.npmjs.com/cli/v7/commands/npm-ci](https://docs.npmjs.com/cli/v7/commands/npm-ci)
- [https://stackoverflow.com/questions/64962960/error-enoent-no-such-file-or-directory-stat-home-dylan-steampath](https://stackoverflow.com/questions/64962960/error-enoent-no-such-file-or-directory-stat-home-dylan-steampath)


package-lock.jsonがないときは、手動でnode_modulesを削除するのがよいかもしれない。

```shell
rm -rf node_modules/
npm install
```

ほかに、プログラムに構文エラーがあるときに起きることがある。この場合、`tsc`でエラー箇所を調べて直す。
