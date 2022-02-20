---
# moved from https://aoirint.hatenablog.com/entry/2019/05/25/080329
title: 'Open JTalk'
date: '2019-05-25 08:03:29'
draft: false
channel: 技術ノート
category: 音声合成
tags:
  - '音声合成'
  - 'Python'
---
# Open JTalk

```python
# Open JTalk Test
# python3
# exec `apt install open-jtalk open-jtalk-mecab-naist-jdic`
# get `mei_normal.htsvoice` from http://www.mmdagent.jp/

import subprocess

p = subprocess.Popen('open_jtalk -x /var/lib/mecab/dic/open-jtalk/naist-jdic -m mei/mei_normal.htsvoice -r 1.0 -ow /dev/stdout', stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)

text = 'こんにちは'
out, err = p.communicate(text.encode('utf-8'))

print(text)
print(err)

# outにwavのバイナリが入ってる。必要に応じてここ変えてね
with open('b.wav', 'wb') as fp:
    fp.write(out)
```

- [subprocess で shell=True でリストを与えたときの挙動 - Qiita](https://qiita.com/yoichi22/items/5afa8b3b39c723acb359)
- [合成音声(Open Jtalk)でwavファイルを作成しないで再生する - Qiita](https://qiita.com/sukesuke/items/be2a4562bd809ccc0fab)
- [python - input directly in process.communicate() in subprocess library - Stack Overflow](https://stackoverflow.com/questions/41479825/input-directly-in-process-communicate-in-subprocess-library)
- [subprocessについて調べたメモ - Qiita](https://qiita.com/HidKamiya/items/b2244fb01715eca33965)
- [Debian Jessie に OpenJtalk を入れてテキストを読み上げてみた - 残しときます（自分用）](http://namotch.hatenablog.com/entry/2015/06/25/225000)
- [日本語音声合成Open JTalkをPythonから実行する - 動かざることバグの如し](http://thr3a.hatenablog.com/entry/20180226/1519619690)
- [mmdagent.jp](http://www.mmdagent.jp/)
- [Open JTalk - HMM-based Text-to-Speech System](http://open-jtalk.sp.nitech.ac.jp/index.php)

### See
- mpg123
