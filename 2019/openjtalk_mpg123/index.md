---
# moved from https://aoirint.hatenablog.com/entry/2019/05/25/082902
title: 'Open JTalk mpg123'
date: '2019-05-25 08:29:02'
draft: false
channel: 技術ノート
category: 音声合成
tags:
  - '音声合成'
  - 'Python'
---
# Open JTalk mpg123

Open JTalkの出力したwavをmpg123で再生したらエラー出た。

```shell
[src/libmpg123/layer1.c:30] error: Illegal bit allocation value.
[src/libmpg123/layer1.c:171] error: Aborting layer I decoding after step one.
```

ffmpegでmp3に変換して再生するとok。mpg123では同じくエラーで変換できなかった。

```shell
ffmpeg -i b.wav b.mp3
mpg123 b.mp3
```

ffmpegのinfo

```
Guessed Channel Layout for Input Stream #0.0 : mono
Input #0, wav, from 'b.wav':
  Duration: 00:00:01.17, bitrate: 768 kb/s
    Stream #0:0: Audio: pcm_s16le ([1][0][0][0] / 0x0001), 48000 Hz, mono, s16, 768 kb/s
```

```
Input #0, mp3, from 'b.mp3':
  Metadata:
    encoder         : Lavf57.56.101
  Duration: 00:00:01.20, start: 0.023021, bitrate: 65 kb/s
    Stream #0:0: Audio: mp3, 48000 Hz, mono, s16p, 64 kb/s
```

```shell
ffmpeg -i b.wav c.wav
```

これもダメ

```
Guessed Channel Layout for Input Stream #0.0 : mono
Input #0, wav, from 'c.wav':
  Metadata:
    encoder         : Lavf57.56.101
  Duration: 00:00:01.17, bitrate: 768 kb/s
    Stream #0:0: Audio: pcm_s16le ([1][0][0][0] / 0x0001), 48000 Hz, mono, s16, 768 kb/s
```

```shell
ffmpeg -i b.mp3 d.wav
```

これもダメ

```
Guessed Channel Layout for Input Stream #0.0 : mono
Input #0, wav, from 'd.wav':
  Metadata:
    encoder         : Lavf57.56.101
  Duration: 00:00:01.17, bitrate: 768 kb/s
    Stream #0:0: Audio: pcm_s16le ([1][0][0][0] / 0x0001), 48000 Hz, mono, s16, 768 kb/s
```

```shell
ffmpeg -i b.wav -ac 2 e.wav
```

ステレオにしてみたけどこれもダメ

```
Guessed Channel Layout for Input Stream #0.0 : stereo
Input #0, wav, from 'e.wav':
  Metadata:
    encoder         : Lavf57.56.101
  Duration: 00:00:01.17, bitrate: 1536 kb/s
    Stream #0:0: Audio: pcm_s16le ([1][0][0][0] / 0x0001), 48000 Hz, stereo, s16, 1536 kb/s
```

うーん...

```shell
import subprocess

p = subprocess.Popen('open_jtalk -x /var/lib/mecab/dic/open-jtalk/naist-jdic -m mei/mei_normal.htsvoice -r 1.0 -ow /dev/stdout', stdin=subprocess.PIPE, stdout=subprocess.PIPE, shell=True)

text = msg
out, _ = p.communicate(text.encode('utf-8'))
wav = out

p = subprocess.Popen('ffmpeg -i pipe:0 -f mp3 pipe:1', stdin=subprocess.PIPE, stdout=subprocess.PIPE, shell=True)
out, _ = p.communicate(wav)

sound = out

p = subprocess.Popen('mpg123 -', stdin=subprocess.PIPE, shell=True)
p.communicate(sound)
```

どう考えても遅くなるから、別の再生プログラムを試した方がいいのかな

- [OpenJTalk · えやみぐさ](https://blog.aoirint.com/entry/2019/openjtalk/)
