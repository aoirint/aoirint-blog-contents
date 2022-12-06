---
# moved from https://aoirint.hatenablog.com/entry/2019/09/27/170100
title: Yolo v3でObject Detectionする（darknet）
date: '2019-09-27T17:01:00+09:00'
draft: false
channel: 技術ノート
category: Python
tags:
- Python
- 機械学習
---
# Yolo v3でObject Detectionする（darknet）

- [https://github.com/pjreddie/darknet](https://github.com/pjreddie/darknet)

```shell
git clone https://github.com/pjreddie/darknet.git
```

## データセットの作成
### mydata.data

```
classes = CLASS_NUM
train = mydata-train.txt
test = mydata-test.txt
names = mydata.names
backup = backup/mydata/
```

各ファイルパスは`darknet`の実行ディレクトリからの相対パス。

### mydata.names

```
category1
category2
category3
...
categoryCLASS_NUM
```

ラベル＝\[クラスの数値表現\]に対応するクラス名（行番号＝ラベル）を記述する。

`mydata-train.txt`、`mydata-test.txt`にはデータセット（訓練データ、テストデータ）に含まれる画像のパスをそれぞれ1画像＝1行で記述する。

データセットの画像と同じ階層に、`IMAGE.jpg`のアノテーション情報として`IMAGE.txt`を配置する必要がある（1画像＝1アノテーションファイル）。

<p><code>mydata-train.txt</code>、<code>mydata-test.txt</code>にはデー<a class="keyword" href="http://d.hatena.ne.jp/keyword/%A5%BF%A5%BB%A5%C3%A5%C8">タセット</a>（訓練データ、テストデータ）に含まれる画像のパスをそれぞれ1画像＝1行で記述する。</p>

### IMAGE.txt

```
label center_x center_y width height
```

カラムはスペース区切りで、BoundingBox1つ＝1行で記述する。

画像中のオブジェクトの種類はlabel（0始まりの数値）で表現する。

画像中のオブジェクトのBoundingBoxは`center_x`、`center_y`、`width`、`height`（オブジェクト中心X座標、オブジェクト中心Y座標、オブジェクト幅、オブジェクト高さ）の4パラメータで表現する。`center_x`、`width`は画像幅で除算、`center_y`、`height`は画像高さで除算し、実数で記述する。


## 設定ファイルの作成

cfgディレクトリ以下のファイルをベースに使う。

`batch`、`subdivision`, `width`, `height`などをメモリサイズなどに応じて変更する。デフォルトでは画像にランダムリサイズがかかるようになっているので、メモリ使用量が変動することに注意。

### Yolo v3

`cfg/yolov3.cfg`をコピーする（`mydata-yolov3.cfg`）。

CLASS_NUM=クラス数

```
# L610, 696, 783
classes=CLASS_NUM

# L603, 689, 776
filters=(CLASS_NUM + 5) * 3
```

#### 例

```
# L610, 696, 783
classes=1

# L603, 689, 776
filters=18
```

### Tiny Yolo v3

`cfg/yolov3-tiny.cfg`をコピーする（`mydata-yolov3-tiny.cfg`）。

CLASS_NUM=クラス数

```
# L135, 177
classes=CLASS_NUM

# L127, 171
filters=(CLASS_NUM + 5) * 3
```

#### 例

```
# L135, 177
classes=1

# L127, 171
filters=18
```


## コマンド

darknetの学習済みモデルをダウンロードする。

```shell
wget https://pjreddie.com/media/files/darknet53.conv.74
```

```shell
# 学習
./darknet detector train mydata.data mydata-yolov3.cfg darknet53.conv.74

# 画像ファイルでテスト（対話型）
./darknet detector test mydata.data mydata-yolov3.cfg backup/mydata/mydata-yolov3_ITER.weights

# 画像ファイルでテスト
./darknet detector test mydata.data mydata-yolov3.cfg backup/mydata/mydata-yolov3_ITER.weights IMAGE_FILE
```

※ ファイルパス・カテゴリ名は（たぶん）ASCIIでないとSegmentation Error吐きます


## Python

`darknet/python/darknet.py`は`libdarknet.so`をctypesでPythonから呼び出すことのできるスクリプトになっているが、Python 2ベースのようなのでPython 3で使うのに便利なインタフェースを作成した。`darknet.py`は`_darknet.py`にリネームした。PIL.Imageの場合にtempfileを使わない改修をしたほうがいいかもしれないが、今回は割愛。

（追記 19/10/27） ※ examples/以下にちゃんとしたサンプルがあるみたいです.

`_darknet.py`の`darknet.so`を指定してる箇所（https://github.com/pjreddie/darknet/blob/master/python/darknet.py#L48）を環境変数化したりすると汎用性上がると思う。

### darknet/python/Darknet.py

```python
import os
import sys

import tempfile

sys.path.append(os.path.dirname(__file__))

from _darknet import *

# 標準出力・標準エラー出力の抑制
def silent(verbose=False):
    def _silent(func):
        def wrapper(*args, **kwargs):
            if not verbose:
                devnull = open(os.devnull, 'w')
                stdout = os.dup(1)
                stderr = os.dup(2)
                os.dup2(devnull.fileno(), 1)
                os.dup2(devnull.fileno(), 2)

            res = func(*args, **kwargs)

            if not verbose:
                os.dup2(stdout, 1)
                os.dup2(stderr, 2)
                devnull.close()

            return res

        return wrapper
    return _silent

class Darknet:
    def __init__(self, meta_file, cfg_file, weights_file, verbose=False):
        @silent(verbose=verbose)
        def _init():
            self.net = load_net(cfg_file.encode('ascii'), weights_file.encode('ascii'), 0)
            self.meta = load_meta(meta_file.encode('ascii'))

        _init()

    # [ name, conf, (x, y, w, h) ]
    # x, y is the center of the object

    def detect_pil(self, img_pil, format='jpg', verbose=False):
        with tempfile.NamedTemporaryFile(suffix='.%s' % os.path.basename(format)) as fp:
            filename = fp.name
            img_pil.save(fp.name)

            return self.detect_file(filename, verbose)

    def detect_file(self, img_file, verbose=False):
        @silent(verbose=verbose)
        def _detect():
            return detect(self.net, self.meta, img_file.encode('ascii'))

        results = _detect()

        ret = []
        for result in results:
            name, conf, box = result
            ret.append((
                name.decode('utf-8'),
                conf,
                box,
            ))

        return ret

if __name__ == "__main__":
    import argparse
    import time

    parser = argparse.ArgumentParser()
    parser.add_argument('meta', type=str) # *.data
    parser.add_argument('cfg', type=str) # *.cfg
    parser.add_argument('weights', type=str) # *.weights
    parser.add_argument('img', type=str)
    parser.add_argument('-v', '--verbose', action='store_true')

    args = parser.parse_args()

    darknet = Darknet(meta_file=args.meta, cfg_file=args.cfg, weights_file=args.weights, verbose=args.verbose)

    ts = time.time()
    # direct
    results = darknet.detect_file(args.img, verbose=args.verbose)

    # PIL example
    # from PIL import Image
    # img_pil = Image.open(args.img)
    # results = darknet.detect_pil(img_pil, verbose=args.verbose)

    elapsed = time.time() - ts

    print('FPS: %f (%f s)' % (1/elapsed, elapsed))

    print('%d boxes found' % len(results))
    for result in results:
        name, conf, box = result
        print(name, conf, box)
```


## 参考

- [https://pjreddie.com/darknet/](https://pjreddie.com/darknet/)
- [Tutorial: Build your own custom real-time object classifier](https://towardsdatascience.com/tutorial-build-an-object-detection-system-using-yolo-9a930513643a)
- [GitHub - AlexeyAB/darknet: Windows and Linux version of Darknet Yolo v3 & v2 Neural Networks for object detection (Tensor Cores are used)](https://github.com/AlexeyAB/darknet)


## 付録

### makeでエラーが出るとき

`Path to libdevice library not specified`：環境変数`PATH`にcuda/binを追加（`export PATH=/usr/local/cuda/bin:$PATH`）

### train/test 分割

ImageFolder形式（クラス別ディレクトリ）、1ディレクトリ形式（クラス問わず同ディレクトリ）に対応。アノテーションデータは別途作成する。

#### SplitImageFolder.py

```python
import os
import random
from tqdm import trange

if __name__ == '__main__':
    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument('indir', type=str)
    parser.add_argument('test_rate', type=float)
    parser.add_argument('--flat-input', action='store_true')
    parser.add_argument('--out-train', type=str, default='train.txt')
    parser.add_argument('--out-test', type=str, default='test.txt')
    args = parser.parse_args()

    current_dir = os.path.realpath(args.indir)

    files = []
    if args.flat_input:
        for imgfile in os.listdir(current_dir):
            imgpath = os.path.join(current_dir, imgfile)
            files.append(imgpath)
    else:
        for catfile in os.listdir(current_dir):
            catpath  = os.path.join(current_dir, catfile)
            for imgfile in os.listdir(catpath):
                imgpath = os.path.join(catpath, imgfile)
                files.append(imgpath)

    test_end = int( len(files) * args.test_rate )
    assert len(files) - test_end > 0, args.test_rate

    print('Data num: %d' % len(files))
    print('Train: %d, Test: %d' % (len(files)-test_end, test_end))

    indexes = list(range(len(files)))
    random.shuffle(indexes)

    with open(args.out_train, 'w') as file_train:
        with open(args.out_test, 'w') as file_test:
            for index in trange(len(files)):
                file_idx = indexes[index]

                imgpath = os.path.realpath(files[file_idx])
                imgfile = os.path.basename(imgpath)
                _, imgext = os.path.splitext(imgfile)
                if imgext == '.txt':
                    continue

                if index < test_end:
                    file_test.write(imgpath + "\n")
                else:
                    file_train.write(imgpath + "\n")
```
