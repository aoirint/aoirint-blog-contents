---
title: 'Chrome 101+ Manifest V3のChrome拡張でHTTPリクエストからOriginヘッダを削除する'
date: '2022-12-07T09:00:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: ブラウザ拡張
tags:
  - ブラウザ拡張
---
# Chrome 101+ Manifest V3のChrome拡張でHTTPリクエストからOriginヘッダを削除する

`declarativeNetRequest`機能を使用します。

```js
chrome.declarativeNetRequest.updateDynamicRules({
  removeRuleIds: [1],
  addRules: [
    {
      id: 1,
      priority: 1,
      action: {
        type: 'modifyHeaders',
        requestHeaders: [
          { 'header': 'Origin', 'operation': 'remove' },
        ],
      },
      condition: {
        urlFilter: '127.0.0.1:50021/*',
        initiatorDomains: [chrome.runtime.id], // chrome-extension://{extension_id}
      },
    },
  ],
})
```
