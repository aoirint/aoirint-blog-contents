---
# moved from https://aoirint.hatenablog.com/entry/2019/07/10/000955
title: ログ監視 Python watchdog（ログローテーション未完成）
date: '2019-07-10T00:09:55+09:00'
draft: false
channel: 技術ノート
category: Python
tags:
- Python
---
# ログ監視 Python watchdog（ログローテーション未完成）

アプリケーションのログファイルを監視するシステムをつくる。

- [ファイルの更新をきっかけにコマンド実行 (python編) - Qiita](https://qiita.com/ksato9700/items/ea4b769d010e8cf1fb0c)
- [ログ監視スクリプト - Qiita](https://qiita.com/hidetzu/items/dd51ea30758bd4c8e521)

上の2つをがっちゃんこしたやつを作った。ファイルの変更監視はwatchdog、読み取りはふつうのIO。

※ うーん、ログローテーション対応が微妙かも

```python
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler
import time
import os
from stat import *

class FileWatchHandler(FileSystemEventHandler):
    def __init__(self, path):
        self.path = os.path.realpath(path)
        self.fp = None
        self.fpos = None

        self.init()

        filesize = os.stat(self.path)[ST_SIZE]
        self.fp.seek(filesize) # ファイル末尾
        self.fpos = filesize

    def init(self):
        fp = self.fp
        if fp:
            fp.close()
        fp = open(self.path, 'r')

        self.fp = fp
        self.fpos = None

    def on_created(self, event):
        if event.src_path == self.path:
            print('reset')
            self.init()

    def on_modified(self, event):
        if event.src_path == self.path:
            print('modified')
            self.tail_f()

    def tail_f(self):
        if self.fpos:
            self.fp.seek(self.fpos)

        while True:
            self.fpos = self.fp.tell()
            line = self.fp.readline()

            if not line:
                break
            self.analyze(line)

    def analyze(self, line):
        # TODO:
        print('!', line)

    def close(self):
        self.fp.close()

path = 'test.txt'

handler = FileWatchHandler(path)
observer = Observer()
observer.schedule(handler, os.path.dirname(os.path.realpath(path)))
observer.start()

try:
    observer.join()
except KeyboardInterrupt:
    pass

observer.stop()
handler.close()

print('closed')
```

※ ちょっと修正してみたけど、やっぱり微妙

```python
    def tail_f(self):
        filesize = os.stat(self.path)[ST_SIZE]
        if self.fpos:
            if self.fpos <= filesize:
                self.fp.seek(self.fpos)
            else:
                self.init()

        while True:
            self.fpos = self.fp.tell()
            line = self.fp.readline()
            
            if not line:
                break
            self.analyze(line)
```

（ログローテーションはともかく、）あとはanalyze関数に新しくappendされた行が入ってくるので、それぞれ解析すればOK。

ログは定期的に分割されて別ファイルに移動、新しいログが入ってきたら新規ファイルになるので、
`on_created`で開き直し（実はseekだけすればいいのかな？）。
このとき、作成と同時に内容が書き込まれると`on_modified`が落としちゃったのでファイル末尾への移動は入れてない。
それからリネームでファイルがやってきても動かない（`on_moved`でいけるけど）。

`observer.join`って、`join`だし別スレッドの終了待ちだと思うので、これでいいかな、と思ったけど公式サンプル（[GitHub - gorakhargosh/watchdog: Python library and shell utilities to monitor filesystem events.](https://github.com/gorakhargosh/watchdog)）は違う...。まあプロセス死んだらなんにせよ止まると願って。

末尾追従動作がけっこう面倒くさいので、すなおに`tail -f`をPopenすればよかったかな...

diffもそのうち使うかも？　今回は関係ないけど

- [difflib --- 差分の計算を助ける — Python 3.7.4 ドキュメント](https://docs.python.org/ja/3/library/difflib.html)
- [Pythonのdifflibモジュールを用いて複数行テキストどうしの差分を取得する - 試験運用中なLinux備忘録・旧記事](https://kakurasan.hatenadiary.jp/entry/20100308/p1)
