---
# moved from https://aoirint.hatenablog.com/entry/2019/09/25/234251
title: Python ログ出力抑制 デコレータ
date: '2019-09-25T23:42:51+09:00'
draft: false
channel: 技術ノート
category: Python
tags:
- Python
---
# Python ログ出力抑制 デコレータ

出力を抑制するデコレータ。

Python 3.7.4、ctypesを使ったライブラリ呼び出しで動作を確認（darknet.py）。

```python
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
```

使い方のイメージ（デコレータなせいで長くなってしまう、デコレータじゃなくて単純にコールバック風に関数を呼び出すラップ関数作ってもいいかも...）。

```python
def func(verbose=False):
    @silent(verbose=verbose)
    def _func():
        cfunc()
    
    funcA()
    _func()
    funcB()
```


## 参考

- [Cの共有ライブラリがPythonの標準出力に印刷されないようにするにはどうすればいいですか？ - コードログ](https://codeday.me/jp/qa/20190107/102213.html)
- [ctypes - How do I prevent a C shared library to print on stdout in python? - Stack Overflow](https://stackoverflow.com/questions/5081657/how-do-i-prevent-a-c-shared-library-to-print-on-stdout-in-python/17954769#17954769)
- [標準出力を黙らせるデコレーター - Qiita](https://qiita.com/mojaie/items/fe2b18d5b9fcab1da97d)
- [Pythonのデコレータについて - Qiita](https://qiita.com/mtb_beta/items/d257519b018b8cd0cc2e)
- [Capturing print output from shared library called from python with ctypes module - Stack Overflow](https://stackoverflow.com/questions/9488560/capturing-print-output-from-shared-library-called-from-python-with-ctypes-module/9489139)
