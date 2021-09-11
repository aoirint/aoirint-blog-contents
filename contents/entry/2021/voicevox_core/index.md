---
title: VOICEVOX CoreをUbuntu/GPUで使う（exampleを動かすまで）
date: '2021-09-11 23:40:00'
draft: false
category: 音声合成
tags:
  - 音声合成
  - Ubuntu
  - VOICEVOX
---
# VOICEVOX CoreをUbuntu/GPUで使う（exampleを動かすまで）

**※ VOICEVOX Coreにより生成した音声の利用にあたっては、VOICEVOX Coreの添付文書・利用規約を必ず確認し、従ってください。**

```
Ubuntu 20.04.3 LTS (Focal Fossa)

Python 3.9.6 (pyenv)

NVIDIA-SMI 470.57.02
Driver Version: 470.57.02
CUDA Version: 11.4

cuDNN 8.2.4
```

## LibTorchのダウンロード
- <https://pytorch.org/>

Stable > Linux > LibTorch > C++/Java > CUDA 11.1 > Download here (cxx11 ABI)

`libtorch-cxx11-abi-shared-with-deps-1.9.0+cu111.zip`を使用した。
圧縮時で2.1GB、展開時で5.1GB。

`~/local/libtorch`以下に、`~/local/libtorch/build-version`のように全ファイルを展開する。


## VOICEVOX Coreのダウンロード
- <https://github.com/Hiroshiba/voicevox_core/releases/tag/0.5.1>

圧縮時・展開時ともに430MB。

`~/local/voicevox_core`以下に、`~/local/voicevox_core/libcore.so`のように全ファイルを展開する。

## .bashrc
```shell
export LIBRARY_PATH="$HOME/local/voicevox_core:$LIBRARY_PATH"
export LIBRARY_PATH="$HOME/local/libtorch/lib:$LIBRARY_PATH"
export LD_LIBRARY_PATH="$HOME/local/voicevox_core:$LD_LIBRARY_PATH"
export LD_LIBRARY_PATH="$HOME/local/libtorch/lib:$LD_LIBRARY_PATH"
```

```shell
source ~/.bashrc
```

## サンプルリポジトリをクローン

- <https://github.com/Hiroshiba/voicevox_core>

```shell
git clone https://github.com/Hiroshiba/voicevox_core.git
cd voicevox_core

# 再現性のためにバージョンを固定、実際は最新版の使用を推奨
git checkout 89d0962ab54269023fe0ec3170c7075744f38702
```

## core.hをコピー
```shell
cp core.h example/python/

cd example/python
```

## パッケージのインストール

```shell
# 依存パッケージのインストール
pip3 install -r requirements.txt
```

### pipを使ったインストール
```shell
# coreモジュール（VOICEVOX Core Pythonライブラリ）のインストール
pip3 install .
```

### distutilsを使ったインストール
```shell
# coreモジュール（VOICEVOX Core Pythonライブラリ）のインストール
python3 setup.py install --record files.txt


# 失敗時
python3 setup.py clean
rm -rf build/ core.cpp
```

`files.txt`にインストールされたファイル一覧が出力される。
アンインストール時は、`files.txt`に列挙されたファイルを手動で削除する。

- <https://qiita.com/orion0616/items/dfe476067e499cca8535>

#### files.txt
```
***/lib/python3.9/site-packages/core.cpython-39-x86_64-linux-gnu.so
***/lib/python3.9/site-packages/core-0.0.0-py3.9.egg-info
```

## サンプルプログラム（run.py）の改変
`core.initialize`の第1引数を`libcore.so`のあるディレクトリのパスに変更する。（末尾のスラッシュは必須）。

```python
    core.initialize("/home/user/local/voicevox_core/", use_gpu)
```

## 改変したサンプルプログラム（run.py）の実行
### 四国めたん
```shell
python3 run.py --use_gpu --text "こんにちは" --speaker_id 0

paplay "./こんにちは-0.wav"
```

### ずんだもん
```shell
python3 run.py --use_gpu --text "こんにちはなのだ" --speaker_id 1

paplay "./こんにちはなのだ-0.wav"
```

## 使用するGPUの変更（複数台のGPUが接続された環境）
CUDAを使うアプリケーション一般に適用できる方法。数値は`nvidia-smi`で確認できるGPU番号とは異なることがあるので注意。

```shell
CUDA_VISIBLE_DEVICES=0 python3 run.py --use_gpu --text "こんにちは" --speaker_id 0
CUDA_VISIBLE_DEVICES=1 python3 run.py --use_gpu --text "こんにちは" --speaker_id 0
```

## その他参考
`LD_LIBRARY_PATH`について調べていたが、コンパイル（`python3 setup.py install`）時に必要（`g++`が見に行くパス）なのは`LIBRARY_PATH`、実行時（`core`モジュールロード時）に必要なのは`LD_LIBRARY_PATH`ということらしかった。

- <https://bettamodoki.hatenadiary.jp/entry/20121121/1353480891>

---

- <https://atmarkit.itmedia.co.jp/flinux/rensai/linuxtips/a115makeerror.html>
- <https://please-sleep.cou929.nu/20080718.html>
- <https://qiita.com/Esfahan/items/0064d845ca6faf7f3d47>
- <https://qiita.com/kazatsuyu/items/5c8d9f539cd925fda007>
