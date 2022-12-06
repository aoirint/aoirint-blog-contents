---
title: 'Chrome 101+ Manifest V3のChrome拡張でHTTPリクエストからOriginヘッダを削除する'
date: '2022-12-07T08:30:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: ブラウザ拡張
tags:
  - ブラウザ拡張
---
# Chrome 101+ Manifest V3のChrome拡張でHTTPリクエストからOriginヘッダを削除する

## 動作環境

- Google Chrome 108
- Manifest V3

## 内容

Chrome拡張から送られるHTTPリクエストについて、
ローカルで動くHTTPサーバ`http://127.0.0.1:8000`のOriginチェックを回避するため、
Originヘッダを削除します。

`declarativeNetRequest`機能を使用します。`manifest.json`の`permissions`に`declarativeNetRequestWithHostAccess`（Chrome 96+）の追記が必要です。

加工するHTTPリクエストを、この拡張機能から送られるHTTPリクエストに絞るため、`initiatorDomains`オプション（Chrome 101+）を使用しています。将来的に、より厳密な検査ができるオプションが追加される可能性があります。

- chrome.declarativeNetRequestのドキュメント: [https://developer.chrome.com/docs/extensions/reference/declarativeNetRequest/](https://developer.chrome.com/docs/extensions/reference/declarativeNetRequest/)

また、CORSを回避するには、`manifest.json`の`host_permissions`に`http://127.0.0.1:8000/*`を追記します。

- <https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/host_permissions>

> [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) and [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) access to those origins without cross-origin restrictions (though not for requests from content scripts, as was the case in Manifest V2).

### background.js

```js
// license: CC0
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
        urlFilter: '127.0.0.1:8000/*',
        initiatorDomains: [chrome.runtime.id], // chrome-extension://{extension_id}
      },
    },
  ],
})
```
