---
# moved from https://aoirint.hatenablog.com/entry/2020/05/22/060000
title: User Style Sheet（Firefox）
date: '2020-05-22 06:00:00'
draft: false
channel: 技術ノート
category: StyleSheet
tags:
- StyleSheet
- Firefox
---
# User Style Sheet（Firefox）

- Firefox 76

---

- <https://diary.palm84.com/entry/20190527/1558967870>

Firefox 69以降デフォルトで無効化されたらしいので、`about:config`から`stylesheets`で検索、`toolkit.legacyUserProfileCustomizations.stylesheets`を`true`にする。

ブラウザ右上の三本線メニューからHelp、Troubleshooting Informationの`Profile Directory`という項目から現在のプロファイルのディレクトリがわかる。

`PROFILE_DIR/chrome`ディレクトリを作成、`PROFILE_DIR/chrome/userContent.css`ファイルを作成。あとはこのCSSがすべてのWebサイトに適用されるのでCSSを書いていくだけ。Firefoxを再起動すると変更が適用される。

```css
body { background: black !important; }
```

- <https://developer.mozilla.org/ja/docs/Web/CSS/@document>
- <http://puppet.asablo.jp/blog/2008/11/25/3974444>

`@-moz-document`で特定のドメインのWebサイトにだけスタイルを適用できる（`@document`では動かなかった）。ドキュメントは`@document`のものを見ればいいのかな（Firefoxにしかこのクエリは実装されてないらしい、CSS 4で検討中？）。ID/クラスが被ってる場合に使えるか。

```css
@-moz-document domain(twitter.com) {
  body { background: black !important; }
}
```

- <https://qiita.com/uzumushi/items/f95f9e89fde2a507d7e8>
