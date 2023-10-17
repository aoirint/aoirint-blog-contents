---
# moved from https://aoirint.hatenablog.com/entry/2020/03/20/161930
title: OpenCV FullScreen Window
date: '2020-03-20T16:19:30+09:00'
draft: false
channel: 技術ノート
category: Python
tags:
- Python
- OpenCV
---
# OpenCV FullScreen Window

- [Python - PythonのOpenCVでフルスクリーン表示｜teratail](https://teratail.com/questions/105621)

```python
import cv2

cv2.namedWindow('screen', cv2.WINDOW_NORMAL)
cv2.setProperty('screen', cv2.WND_PROP_FULLSCREEN, cv2.WINDOW_FULLSCREEN)
```

```plain
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.
```

- [python-opencvでQt plugin "cocoa"が見つからないというエラー - Qiita](https://qiita.com/jmtsn/items/aa73d837b3a8d4158885)

## 黒画面

```python
import cv2
import numpy as np

# Change here
WIDTH = 1920
HEIGHT = 1080

# For secondary monitor,
LEFT = 0
TOP = 0

screen = np.zeros((HEIGHT, WIDTH), dtype=np.uint8)
cv2.namedWindow('screen', cv2.WINDOW_NORMAL)
cv2.moveWindow('screen', LEFT, TOP)
cv2.setWindowProperty('screen', cv2.WND_PROP_FULLSCREEN, cv2.WINDOW_FULLSCREEN)

cv2.imshow('screen', screen)

cv2.waitKey(0)
```

For dynamic image size,

- [https://stackoverflow.com/questions/3129322/how-do-i-get-monitor-resolution-in-python](https://stackoverflow.com/a/31171430)
- [macos - How to get the desktop resolution in Mac via Python? - Stack Overflow](https://stackoverflow.com/questions/1281397/how-to-get-the-desktop-resolution-in-mac-via-python)

### クリック

```python
def on_click(event, x, y, flags, param):
    if event == cv2.EVENT_LBUTTONDOWN:
        pass

cv2.setMouseCallback('screen', on_click)
```

- [ペイントツールとしてのマウス — OpenCV-Python Tutorials 1 documentation](http://labs.eecs.tottori-u.ac.jp/sd/Member/oyamada/OpenCV/html/py_tutorials/py_gui/py_mouse_handling/py_mouse_handling.html)
