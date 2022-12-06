---
# moved from https://aoirint.hatenablog.com/entry/2017/09/24/110338
title: 'VS2017でOpenCVを使う（Win pack）'
date: '2017-09-24T11:03:38+09:00'
draft: false
channel: 技術ノート
category: OpenCV
tags:
  - OpenCV
  - 'Visual Studio'
  - Windows
---
# VS2017でOpenCVを使う（Win pack）

## What

Visual Studio 2017でOpenCVを使いたい。

## Environment

- Windows 10 Home
- Visual Studio 2017

## How

### OpenCVのダウンロード

- [OpenCV library](http://opencv.org/)

OpenCV libraryのReleasesから最新の"Win pack"をダウンロード。

"Win pack"は自己展開exeになっているので、適当なディレクトリを指定して展開。

### VSプロジェクトの設定

新規にプロジェクトを作る場合、Visual C++から空のプロジェクトを作成。

ソリューションエクスプローラからプロジェクトのプロパティを開き、"VC++ディレクトリ"のツリーに移る。

"インクルードディレクトリ"、"ライブラリディレクトリ"を編集し、展開ディレクトリ以下の`build\include`をそれぞれに追加する。

"リンカー"のツリーに移り、"追加のライブラリディレクトリ"を編集し、展開ディレクトリ以下の`build\x64\vc14\lib`を追加する。

"リンカー"のツリーから、"入力"を選択し、"追加の依存ファイル"に必要なライブラリ（.lib）を追加する。Win packには`opencv_world330d.lib`（デバッグ）と`opencv_world330.lib`（リリース）しかないので、構成に対応する方を追加。

### PATHの設定

展開ディレクトリ以下の`build\x64\vc14\bin`にPATHを通す。

### サンプル

```cpp
#include <iostream>
#include <opencv2\opencv.hpp>

using namespace cv;

int main() {
  Mat mat(400, 400, CV_8UC3, Scalar(255, 128, 0));

  imshow("mat", mat);

  waitKey(0);

  return 0;
}
```

水色のウインドウが表示されればOK。ウインドウをアクティブにして何かキーを押せば終了する。
