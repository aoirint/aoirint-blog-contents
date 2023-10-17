---
title: nginx, リバースプロキシからのReferrer-Policyヘッダを置換する
date: '2022-07-01T09:00:00+09:00'
draft: false
channel: 技術ノート
category: nginx
tags:
  - nginx
---
# nginx, リバースプロキシからのReferrer-Policyヘッダを置換する

```nginx
add_header Referrer-Policy no-referrer;
# add_header Referrer-Policy same-origin;

proxy_hide_header Referrer-Policy; # remove Referrer-Policy from original response
```

- [https://takuya-1st.hatenablog.jp/entry/2018/11/01/040000](https://takuya-1st.hatenablog.jp/entry/2018/11/01/040000)
- [https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Referrer-Policy](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Referrer-Policy)
- [https://stackoverflow.com/a/55692346](https://stackoverflow.com/a/55692346)

## Mastodon

Mastodonでリファラを送らないようにしたかった。

HTMLタグのmetaタグを使う方法をよく使うのだけれど、
ブログとかによくあるheadタグへの任意HTML追加機能のようなものは
Mastodon v3.5.3にはなさそうだった。

- [https://github.com/mastodon/mastodon/issues/9183](https://github.com/mastodon/mastodon/issues/9183)

同一オリジンでもリファラを送らないようにする（no-referrer）とMastodonの一部の設定が更新できなくなくなるようなので、
Mastodonでは`Referrer-Policy: same-origin`に設定した。

単純に`add_header`するだけだと、Mastodonが返す`Referrer-Policy`とnginxが追加する`Referrer-Policy`で2つのヘッダになってしまった。

```plain
referrer-policy: origin
referrer-policy: no-referrer
```

そこで、

```nginx
proxy_hide_header Referrer-Policy;
```

でMastodonから返ってきたReferrer-Policyを削除するようにした。

あとは、

```nginx
add_header Referrer-Policy same-origin;
```

するだけ。

- [https://mstdn.aoirint.com/@aoirint/108569086590516035](https://mstdn.aoirint.com/@aoirint/108569086590516035)
