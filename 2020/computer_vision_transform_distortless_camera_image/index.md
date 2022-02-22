---
# moved from https://aoirint.hatenablog.com/entry/2020/03/21/044833
title: 歪みなしカメラ画像の座標変換
date: '2020-03-21 04:48:33'
draft: false
channel: 技術ノート
category: ComputerVision
tags:
- ComputerVision
---
# 歪みなしカメラ画像の座標変換

- [OpenCV: Camera Calibration and 3D Reconstruction](https://docs.opencv.org/4.2.0/d9/d0c/group__calib3d.html#details)


### カメラ行列（内部パラメータ / intrinsic parameters）

$$
A = \left(
\begin{array}{ccc}
f_x & 0 & c_x \\
0 & f_y & c_y \\
0 & 0 & 1
\end{array}
\right)
$$

- 焦点距離 \( f_x, f_y \)
    - 単位は画素（pixels）
    - 2つの役割をもつ
        - ピンホールカメラモデルにおける焦点距離\(tex: f \)（参考：[ピンホールカメラモデル](https://blog.aoirint.com/entry/2020/computer_vision_pinhole_camera_model/)）
        - 任意の単位で表される変換前の座標からコンピュータ上の画素の単位であるピクセル単位に変換する
    - ピンホールカメラモデルでは1つの焦点距離\(tex: f \)しかなかったのに、なぜ\(tex: f_x, f_y \)と2つ含まれているのか
        - 2つ目の役割（撮像素子の位置＝実世界の距離からピクセル単位に変換）のため、撮像素子の特性（px/mm）がx、y方向で異なる場合を想定している（\( f_x, f_y \)はレンズの物理的な焦点距離 \( tex: f \)（mm）と撮像素子の特性\( c_x, c_y\)の積。また、カメラ画像からそれぞれを独立に求めることはできない）
        - 参考：[computer vision - Why does the focal length in the camera intrinsics matrix have two dimensions? - Stack Overflow](https://stackoverflow.com/questions/16329867/why-does-the-focal-length-in-the-camera-intrinsics-matrix-have-two-dimensions), [Learning OpenCV](https://books.google.ch/books?id=seAgiOfu2EIC&pg=PA373)
- 画像の中心 \(c_x, c_y\)
    - 単位は画素（pixels）


### 射影変換行列（Perspective transform matrix, 外部パラメータ/ extrinsic parameters）

#### 参考：2次元の射影変換行列

$$
[R|t] = \left(
\begin{array}{cccc}
r_{11} & r_{12} & r_{13} & t_x \\
r_{21} & r_{22} & r_{23} & t_y \\
\end{array}
\right)
$$

#### 3次元の射影変換行列

$$
[R|t] = \left(
\begin{array}{cccc}
r_{11} & r_{12} & r_{13} & t_x \\
r_{21} & r_{22} & r_{23} & t_y \\
r_{31} & r_{32} & r_{33} & t_z \\
\end{array}
\right)
$$


### 実世界の位置とカメラ画像座標の射影変換

#### 「任意の原点・角度の位置（世界座標）」と「カメラの位置、角度からの相対座標（カメラ座標）」の変換

$$
P_c =
\left(
\begin{array}{c}
x \\
y \\
z
\end{array}
\right)
= [R|t] \left(
\begin{array}{c}
X_w \\
Y_w \\
Z_w \\
1
\end{array}
\right)
$$

#### 「カメラの位置、角度からの相対座標（カメラ座標）」と「カメラ画像中の画素の座標」の変換

$$
P_i =
\left(
\begin{array}{c}
u \\
v \\
1
\end{array}
\right)
= A\ P_c
= A\ [R|t] \left(
\begin{array}{c}
X_w \\
Y_w \\
Z_w \\
1
\end{array}
\right)
$$
