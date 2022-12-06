---
title: npm startで構文エラーが消えないときの対処（React）
date: '2021-06-05T01:30:00+09:00'
updated: '2021-08-22T22:30:00+09:00'
draft: false
channel: 技術ノート
category: React
tags:
  - React
  - Node.js
  - npm
---

# npm startで構文エラーが消えないときの対処（React）

npm startで構文エラーが消えなくなることがある。

npm startでサーバを起動してコードの変更を監視したままの状態で、gitでブランチを切り替えたときキャッシュが正しく更新されないことが原因と思われる。

キャッシュを破棄するには、

```shell
npm start -- --reset-cache
```

依存関係が変わっているとエラーの原因になるので、モジュールもインストールし直すのがよい。

```shell
npm ci
```

- [https://qiita.com/kt3k/items/758c4f3595ffd7e74f9f](https://qiita.com/kt3k/items/758c4f3595ffd7e74f9f)
