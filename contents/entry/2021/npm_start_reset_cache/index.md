---
canonical_url: ./
title: npm startで構文エラーが消えないときの対処
date: '2021-06-05 01:30:00'
draft: false
category: Node.js
tags:
  - Node.js
  - npm
---

# npm startで構文エラーが消えないときの対処

npm startで構文エラーが消えなくなることがある。

npm startでサーバを起動してコードの変更を監視したままの状態で、gitでブランチを切り替えたときキャッシュが正しく更新されないことが原因と思われる。

キャッシュを破棄するには、

```shell
npm start -- --reset-cache
```

- [https://qiita.com/kt3k/items/758c4f3595ffd7e74f9f](https://qiita.com/kt3k/items/758c4f3595ffd7e74f9f)
