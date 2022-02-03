---
title: VOICEVOX CoreをUbuntu/GPUで使う（exampleを動かすまで）
date: '2021-09-11 23:40:00'
updated: '2022-02-03 10:30:00'
draft: false
category: 音声合成
tags:
  - 音声合成
  - Ubuntu
  - VOICEVOX
---
# VOICEVOX CoreをUbuntu/GPUで使う（exampleを動かすまで）

**※ VOICEVOX Coreにより生成した音声の利用にあたっては、VOICEVOX Coreの添付文書・利用規約を必ず確認し、従ってください。**

- <https://github.com/Hiroshiba/voicevox_core>

```
Ubuntu 20.04.3 LTS (Focal Fossa)

Python 3.9.6 (pyenv)

NVIDIA-SMI 470.57.02
Driver Version: 470.57.02
CUDA Version: 11.4

cuDNN 8.2.4
```

## VOICEVOXの音声合成の仕組みについて

VOICEVOXの音声合成の仕組みについては、以下の記事が参考になるかもしれません。

- [VOICEVOXの音声合成エンジンの紹介 | Hiho's Blog](https://blog.hiroshiba.jp/voicevox-engine-introduction/)

VOICEVOX Coreを効果的に使うには、ある程度音声合成の専門知識が必要になると思います。

簡易にTTSを利用したい場合には、VOICEVOX EngineのHTTP APIを使うことをおすすめします。

- [voicevox_engine API Document](https://voicevox.github.io/voicevox_engine/api/)

VOICEVOX Engineは、公式Dockerイメージが公開されています。

- <https://github.com/Hiroshiba/voicevox_engine>
- <https://hub.docker.com/r/hiroshiba/voicevox_engine>

### VOICEVOX Coreのインタフェースについて

<details>

わたしは音声合成については素人ですが、参考のため現在の理解を書いておきます。

VOICEVOX Coreでは、
「音素ごとの長さの推定 `replace_phoneme_length / yukarin_s_forward`」、
「モーラごとの音高の推定 `replace_mora_pitch / yukarin_sa_forward`」、
「音声波形の推定 `synthesis / decode_forward`」の3つの深層学習モデルを使って音声合成します。

OpenJTalkには、辞書に基づいてテキストを解析し、アクセント句・モーラ・アクセント位置・疑問文フラグなどの情報で構成された
フルコンテキストラベルというデータに変換する機能があります。

フルコンテキストラベルの仕様は、以下のURLを開き、
`HTS-2.3 > Speaker dependent training demo > Japanese > tar.bz2`から`HTS-demo_NIT-ATR503-M001.tar.bz2`をダウンロード・展開し、
`data/lab_format.pdf`を見ると英語で記述されています。

- <https://hts.sp.nitech.ac.jp/?Download#u879c944>
- <https://twitter.com/hiho_karuta/status/1059845813143138304>

VOICEVOX Engineでは、フルコンテキストラベルやVOICEVOX Coreを使って、
テキストを調声用のデータ構造（AudioQuery）に変換し、
また調声用のデータ構造を音声（Wav）に変換します。

VOICEVOXエディタでは、調声用のデータ構造を操作して、アクセント位置、音高（日本語ではイントネーションと同じらしい）、音素長を調声するUIなどが実装されています。

</details>

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
