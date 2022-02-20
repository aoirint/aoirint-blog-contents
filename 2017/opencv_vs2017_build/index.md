---
# moved from https://aoirint.hatenablog.com/entry/2017/09/24/123203
title: 'VS2017でOpenCVをビルドする'
date: '2017-09-24 12:32:03'
draft: false
channel: 技術ノート
category: OpenCV
tags:
  - 'OpenCV'
  - 'C++'
  - 'Visual Studio'
  - Windows
---
## What
Visual Studio 2017でOpenCVをビルドしたい。

## Environment
- Windows 10 Home
- Visual Studio 2017
- CMake 3.9.3

## How
### Download Sources
```shell
$ git clone https://github.com/opencv/opencv.git
```

または

- [OpenCV library](http://opencv.org/)

### CMake

- [CMake](https://cmake.org/)

CMake-GUIを起動、上部のテキストフィールドにソースのパスと出力先パスを入力する。
ソースは`git clone`してきたならopencvフォルダ、`CMakeLists.txt`のあるところ。出力先は適当に`build`とか。

Configureを押して`Visual Studio 15 2017 Win64`を選択、"Finish"。設定が始まる。

終わったら"Generate"。出力先に`OpenCV.sln`が生成される。

17/11/05追記："Generate"の前に`BUILD_*`の設定を変えると出力されるlibが変わる。

### Visual Studio

`OpenCV.sln`を開いて、構成をReleaseに変えて`CMakeTargets/INSTALL`をビルド。

17/11/05追記：`_d`のつくデバッグ用ライブラリを生成したいならDebug構成でビルドすればよい。

正常に終われば、出力先の`install\x64\vc15\lib`に`opencv_core330.lib`などが出力されている。

- [VS2017でOpenCVを使う（Win pack）](https://blog.aoirint.com/entry/2017/opencv_vs2017/)

`install\x64\vc15\bin`にPATHを通し、上のページに従ってVSプロジェクトを設定（ライブラリは適宜必要なもの）する。

## Reference
- [OpenCV: Installation in Windows](http://docs.opencv.org/3.3.0/d3/d52/tutorial_windows_install.html)
