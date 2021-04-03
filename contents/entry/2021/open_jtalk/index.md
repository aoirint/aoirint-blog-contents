---
canonical_url: ./
title: Open JTalkを試す
# og_image:
# twitter_card: summary_large_image
# og_description: テンプレート
date: '2021-04-04 08:00:00'
draft: false
category: 音声合成
tags:
  - 音声合成
  - Open JTalk
  - Docker
---

# Open JTalkを試す

以前にもOpen JTalkを触っていたが、時間が経った＆記事が雑だったのでDockerでまとめ直しておく。

- [Open JTalk - aoirint's note](https://aoirint.hatenablog.com/entry/2019/05/25/080329)

Dockerfileを作る。

```dockerfile
FROM ubuntu:bionic

RUN apt-get update && \
    apt-get install -y \
        open-jtalk \
        open-jtalk-mecab-naist-jdic \
        hts-voice-nitech-jp-atr503-m001

WORKDIR /data
ENTRYPOINT [ "open_jtalk" ]
```

ビルドする。

```shell
docker build . -t aoirint/open_jtalk
```

実行する。
```shell
echo "こんにちは" >> text.txt

# テキストファイルの文字列から音声を生成
docker run --rm -v "$PWD:/data" aoirint/open_jtalk -x /var/lib/mecab/dic/open-jtalk/naist-jdic/ -m /usr/share/hts-voice/nitech-jp-atr503-m001/nitech_jp_atr503_m001.htsvoice text.txt -ow output.wav

# 標準入力の文字列から音声を生成
cat text.txt | docker run --rm -i -v "$PWD:/data" aoirint/open_jtalk -x /var/lib/mecab/dic/open-jtalk/naist-jdic/ -m /usr/share/hts-voice/nitech-jp-atr503-m001/nitech_jp_atr503_m001.htsvoice -ow output.wav

# 標準入力の文字列から音声を生成して、標準出力に書き出す（wav形式）
cat text.txt | docker run --rm -i -v "$PWD:/data" aoirint/open_jtalk -x /var/lib/mecab/dic/open-jtalk/naist-jdic/ -m /usr/share/hts-voice/nitech-jp-atr503-m001/nitech_jp_atr503_m001.htsvoice -ow /dev/stdout > output.wav
```

mmdagent.jpで配布されているボイスファイル（htsvoice）を使う。

- [MMDAgent - Browse /MMDAgent_Example at SourceForge.net](https://sourceforge.net/projects/mmdagent/files/MMDAgent_Example/)

ここから最新のMMDAgent Exampleをダウンロードする。
`Voice/mei/mei_normal.htsvoice`に目的のボイスファイルがある（CC BY 3.0）。
これを作業ディレクトリ（`$PWD`）にコピーしておく。

```shell
# 他のhtsvoiceを使う（mei_normal.htsvoice）
docker run --rm -v "$PWD:/data" aoirint/open_jtalk -x /var/lib/mecab/dic/open-jtalk/naist-jdic/ -m /data/mei_normal.htsvoice text.txt -ow output.wav
```


ヘルプはこのようになっている。

```shell
$ docker run --rm -it -v "$PWD:/data" aoirint/open_jtalk -h
The Japanese TTS System "Open JTalk"
Version 1.10 (http://open-jtalk.sourceforge.net/)
Copyright (C) 2008-2016 Nagoya Institute of Technology
All rights reserved.

The HMM-Based Speech Synthesis Engine "hts_engine API"
Version 1.10 (http://hts-engine.sourceforge.net/)
Copyright (C) 2001-2015 Nagoya Institute of Technology
              2001-2008 Tokyo Institute of Technology
All rights reserved.

Yet Another Part-of-Speech and Morphological Analyzer "Mecab"
Version 0.996 (http://mecab.sourceforge.net/)
Copyright (C) 2001-2008 Taku Kudo
              2004-2008 Nippon Telegraph and Telephone Corporation
All rights reserved.

NAIST Japanese Dictionary
Version 0.6.1-20090630 (http://naist-jdic.sourceforge.jp/)
Copyright (C) 2009 Nara Institute of Science and Technology
All rights reserved.

open_jtalk - The Japanese TTS system "Open JTalk"

  usage:
       open_jtalk [ options ] [ infile ] 
  options:                                                                   [  def][ min-- max]
    -x  dir        : dictionary directory                                    [  N/A]
    -m  htsvoice   : HTS voice files                                         [  N/A]
    -ow s          : filename of output wav audio (generated speech)         [  N/A]
    -ot s          : filename of output trace information                    [  N/A]
    -s  i          : sampling frequency                                      [ auto][   1--    ]
    -p  i          : frame period (point)                                    [ auto][   1--    ]
    -a  f          : all-pass constant                                       [ auto][ 0.0-- 1.0]
    -b  f          : postfiltering coefficient                               [  0.0][ 0.0-- 1.0]
    -r  f          : speech speed rate                                       [  1.0][ 0.0--    ]
    -fm f          : additional half-tone                                    [  0.0][    --    ]
    -u  f          : voiced/unvoiced threshold                               [  0.5][ 0.0-- 1.0]
    -jm f          : weight of GV for spectrum                               [  1.0][ 0.0--    ]
    -jf f          : weight of GV for log F0                                 [  1.0][ 0.0--    ]
    -g  f          : volume (dB)                                             [  0.0][    --    ]
    -z  i          : audio buffer size (if i==0, turn off)                   [    0][   0--    ]
  infile:
    text file                                                                [stdin]
```

- [Debian Jessie に OpenJtalk を入れてテキストを読み上げてみた - 残しときます（自分用）](http://namotch.hatenablog.com/entry/2015/06/25/225000)
- [合成音声(Open Jtalk)でwavファイルを作成しないで再生する - Qiita](https://qiita.com/sukesuke/items/be2a4562bd809ccc0fab)
- [mmdagent.jp](http://www.mmdagent.jp/)
- [Open JTalk - HMM-based Text-to-Speech System](http://open-jtalk.sp.nitech.ac.jp/index.php)
