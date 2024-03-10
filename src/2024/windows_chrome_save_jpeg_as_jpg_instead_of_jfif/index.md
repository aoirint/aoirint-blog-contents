---
title: 'Windows 11, ChromeでJPEG画像を保存するときの拡張子をjfifからjpgに変更する'
date: '2024-03-10T19:20:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Windows
tags:
  - Windows
---
# Windows 11, ChromeでJPEG画像を保存するときの拡張子をjfifからjpgに変更する

- Windows 11

Windows版のGoogle Chromeで`image/jpeg`の画像をダウンロードしようとすると、ファイル名の拡張子が`.jfif`として保存される場合がある。
他のアプリケーションやブラウザ、一部のWebサイト、OSでは`image/jpeg`が`.jpg`で保存される場合もあり、同じ形式にもかかわらず拡張子が混在してしまうため、管理が面倒になる。
また、`.jfif`拡張子をJPEG画像として認識することに対応してないプログラムがプレビューを生成しなかったり、
Webアプリケーションがファイルの種類の判別に失敗してContent-Typeを`application/octet-stream`として扱い、リンクをクリックしたときにブラウザで開くのではなくダウンロードされるなど、不便になることがある。

`regedit`を使って以下のレジストリ値を書き換えることで、`image/jpeg`の画像をダウンロードするときの拡張子を変更できる。予期しない影響が出る可能性があるため、変更する場合は注意すること。

- `HKEY_CURRENT_USER\Software\Classes\MIME\Database\Content Type\image/jpeg`

## 拡張子をjpgに変更

以下の内容を拡張子`.reg`のテキストファイルとして保存し、実行することで反映できる。

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Classes\MIME\Database\Content Type\image/jpeg]
"Extension"=".jpg"
```

## 拡張子をjfifに戻す

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Classes\MIME\Database\Content Type\image/jpeg]
"Extension"=".jfif"
```

## 参考

- [ブラウザでJPG画像を保存しようとすると.jfifの拡張子で保存されるのを.jpgに戻す方法 - [その他] ぺんたん info](https://pentan.info/program/jfif2jpg.html)
