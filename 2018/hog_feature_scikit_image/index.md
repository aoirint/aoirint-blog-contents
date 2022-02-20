---
# moved from https://aoirint.hatenablog.com/entry/2018/12/15/021847
title: 'HOG特徴量の計算（Scikit-image）'
date: '2018-12-15 02:18:47'
draft: false
channel: 技術ノート
category: 画像処理
tags:
  - '画像処理'
  - 'Python'
---
# HOG特徴量の計算（Scikit-image）

```shell
pip install scikit-image
```

```python
from skimage.feature import hog
import numpy as np

size = 32
channel = 1
image = np.random.randn(size, size, channel)
feature = hog(image)

print(feature.shape)
print(feature)
```

デフォルト（勾配方向数9, セルサイズ8, ブロックサイズ3）では32x32の画像に対して324次元のHOG特徴量が出力される。

## 出力例

```
(324,)
[0.01877925 0.00125122 0.00899619 0.00630656 0.01330613 0.0150924
...
0.00763393 0.01668535 0.01852621 0.01204107 0.00433798 0.01147866]
```

- [画像からHOG特徴量の抽出 - Qiita](https://qiita.com/mikaji/items/3e3f85e93d894b4645f7)
  - <https://web.archive.org/web/20170917002713/http://qiita.com/mikaji/items/3e3f85e93d894b4645f7>
- ドキュメント： <https://scikit-image.org/docs/dev/api/skimage.feature.html#skimage.feature.hog>
