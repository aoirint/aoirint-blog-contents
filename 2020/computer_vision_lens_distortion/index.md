---
# moved from https://aoirint.hatenablog.com/entry/2020/03/21/051834
title: Lens Distortion
date: '2020-03-21T05:18:34+09:00'
draft: false
channel: 技術ノート
category: 画像処理
tags:
  - 画像処理
  - 'Computer Vision'
---
# Lens Distortion

- [OpenCV: Camera Calibration and 3D Reconstruction](https://docs.opencv.org/4.2.0/d9/d0c/group__calib3d.html#details)
- [歪みなしカメラ画像の座標変換 - えやみぐさ](https://blog.aoirint.com/entry/2020/computer_vision_transform_distortless_camera_image/)

カメラ座標

$$
P_c = \left(
\begin{array}{c}
X_c \\
Y_c \\
Z_c
\end{array}
\right)
$$

$$
x' = X_c / Z_c \\
y' = Y_c / Z_c \\
r^2 = x'^2 + y'^2
$$

歪み補正 / Distortion correction

$$
x'' = x' \frac{ 1 + k_1\ r^2 + k_2\ r^4 + k_3\ r^6 }{ 1 + k_4\ r^2 + k_5\ r^4 + k_6\ r^6 } + 2 p_1 x' y' + p_2 (r^2 + 2 x'^2) + s_1 r^2 + s_2 r^4 \\
y'' = y' \frac{ 1 + k_1\ r^2 + k_2\ r^4 + k_3\ r^6 }{ 1 + k_4\ r^2 + k_5\ r^4 + k_6\ r^6 } + p_1 (r^2 + 2 y'^2) + 2 p_2 x' y' + s_1 r^2 + s_2 r^4
$$

カメラ座標変換

$$
\left(
\begin{array}{c}
u \\
v
\end{array}
\right)
=

\left(
\begin{array}{ccc}
f_x & 0 & c_x \\
0 & f_y & c_y \\
\end{array}
\right)

\left(
\begin{array}{c}
x'' \\
y'' \\
1
\end{array}
\right)
$$
