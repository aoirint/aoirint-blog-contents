---
# moved from https://aoirint.hatenablog.com/entry/2017/09/24/091510
title: 'エクスプローラからMSYS2を開く + メニュー項目の追加'
date: '2017-09-24T09:15:10+09:00'
draft: false
channel: 技術ノート
category: MSYS2
tags:
  - MSYS2
  - Windows
---
# エクスプローラからMSYS2を開く + メニュー項目の追加

## What

MSYS2をエクスプローラから開きたい

## Environment

Windows 10 Home

## How

### u_msys2.reg

```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\Background\shell\u_msys2]
@="MSYS2 Here"

[HKEY_CLASSES_ROOT\Directory\Background\shell\u_msys2\command]
@="C:\\msys64\\msys2.exe"
```

上のようにレジストリキーを追加する。

```ini
#CHERE_INVOKING=1
#MSYS2_PATH_TYPE=inherit
```

`msys2.ini`で上のコメントアウトを外し、起動時にパスがエクスプローラ側のフォルダになるようにする（どこから持ってきてるんだ？　作業フォルダ？）。

## Appendix

エクスプローラの右クリックメニューに任意の項目を追加する

以下のレジストリキーに特定のサブキーを追加する。

|対象|キー名|
|:--|:--|
|フォルダの背景|`HKEY_CLASSES_ROOT\Directory\Background\shell`|
|フォルダ|`HKEY_CLASSES_ROOT\Folder\shell`|
|任意のファイル|`HKEY_CLASSES_ROOT\*\shell`|
|ファイル `*.hoge`|`HKEY_CLASSES_ROOT\.hoge\shell`|

追加するサブキーは`hoge`、`hoge\command`の2つ。hogeには自由な名前を付けられるっぽい。

`hoge`には、既定値にメニューの名称を設定。

`hoge\command`には、既定値にアプリケーションパスを設定。これは、PATHを通していても無効かもしれない。`%V`は"メニューを開いた"、つまり選択したパスで置換される。

### Sample

以下はフォルダの右クリックメニューに`cmd.exe /k dir`を実行する項目"TEST"を追加するサンプル。

### u_dir.reg

```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Folder\shell\u_dir]
@="TEST"

[HKEY_CLASSES_ROOT\Folder\shell\u_dir\command]
@="cmd.exe /k dir"
```

こちらは同キーを削除する。 !u_dir.reg

```reg
Windows Registry Editor Version 5.00

[-HKEY_CLASSES_ROOT\Folder\shell\u_dir]
```

## Reference

- [作業メモ：msys2インストール、エクスプローラーから起動 - milk_spoonのブログ](http://spoonblog.hatenablog.com/entry/2017/03/01/215247)
- [エクスプローラの右クリックメニューをカスタマイズする - Qiita](https://qiita.com/tueda/items/0036ee8e9280f70f04f0)
- [.reg ファイルを使用してレジストリ サブキーおよび値を追加、変更または削除する方法](https://support.microsoft.com/ja-jp/help/310516/)
- [Windowsのシンボリックリンクとジャンクションとハードリンクの違い：Tech TIPS - ＠IT](http://www.atmarkit.co.jp/ait/articles/1306/07/news111.html)
