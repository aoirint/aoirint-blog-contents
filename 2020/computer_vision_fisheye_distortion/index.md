---
# moved from https://aoirint.hatenablog.com/entry/2020/03/21/053306
title: Fisheye distortion
date: '2020-03-21T05:33:06+09:00'
draft: false
channel: 技術ノート
category: 画像処理
tags:
  - 画像処理
  - 'Computer Vision'
  - OpenCV
---
# Fisheye distortion

- [OpenCV: Camera Calibration and 3D Reconstruction](https://docs.opencv.org/4.2.0/d9/d0c/group__calib3d.html#details)
- [OpenCV: Fisheye camera model](https://docs.opencv.org/4.2.0/db/d58/group__calib3d__fisheye.html#details)
- [歪みなしカメラ画像の座標変換 - えやみぐさ](https://blog.aoirint.com/entry/2020/computer_vision_transform_distortless_camera_image/)

## カメラ座標

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
r^2 = x'^2 + y'^2 \\
\theta = {\rm atan}(r)
$$

## 歪み補正 / Distortion correction

$$
\theta_d = \theta ( 1 + k_1 \theta^2 + k_2 \theta^4 + k_3 \theta^6 + k_4 \theta^8 )\\
x'' = \frac{ \theta_d }{r} x' \\
y'' = \frac{ \theta_d }{r} y'
$$

## カメラ座標変換

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

## OpenCV

```python
K = np.array([
    [ 1, 0, 320 ],
    [ 0, 1, 240 ],
    [ 0, 0, 1 ],
], dtype=np.float32)
D = np.array([ 1, 0, 0, 0 ], dtype=np.float32)

undistorted_points = np.array([[
    [ 0, 0 ],
    [ 0, 10 ],
    [ 10, 0 ],
    [ 10, 10 ],
]], dtype=np.float32)

distorted_points = cv2.fisheye.distortPoints(undistorted_points, K, D)

distorted_points
```

```plain
array([[[320.     , 240.     ],
        [320.     , 244.65497],
        [324.65497, 240.     ],
        [323.44827, 243.44826]]], dtype=float32)
```
