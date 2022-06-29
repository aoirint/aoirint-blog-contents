---
title: MSFS2020の追加ダウンロードファイルを別ディスクに移動する
date: '2022-06-29 14:30:00'
draft: false
channel: 雑記
category: ゲーム
tags:
  - ゲーム
  - MSFS
---
# MSFS2020の追加ダウンロードファイルを別ディスクに移動する

- Microsoft Flight Simulator Game of the Year Edition (MSFS2020)
  - <https://store.steampowered.com/app/1250410/Microsoft_Flight_Simulator_Game_of_the_Year_Edition/>

MSFSはSteamライブラリで見ると1GBくらいだけれど、
起動後に100GB以上の追加ファイルをダウンロードする。

これがメインディスクを圧迫するので、多少のパフォーマンス悪化を覚悟してサブディスクに移動する。

`%APPDATA%\Microsoft Flight Simulator\UserCfg.opt`を開く。末尾の

```
InstalledPackagesPath "C:\Users\myuser\AppData\Roaming\Microsoft Flight Simulator\Packages"
```

を好きなディレクトリに書き換える。

```
InstalledPackagesPath "D:\Microsoft Flight Simulator\Packages"
```

あとは実体をエクスプローラで移動させればOK。

## 参考

- <https://jp.fl510.aero/index.php/msfs/blog/msfs2020-known-issues-list>
