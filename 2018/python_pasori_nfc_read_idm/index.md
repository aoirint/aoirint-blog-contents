---
# moved from https://aoirint.hatenablog.com/entry/2018/08/30/164619
title: 'PythonからPaSoRiを使って交通系ICカードのIDmを読む'
date: '2018-08-30T16:46:19+09:00'
draft: false
channel: 技術ノート
category: NFC
tags:
  - 'NFC'
  - 'Python'
---
# PythonからPaSoRiを使って交通系ICカードのIDmを読む

## 環境

- Ubuntu 18.04 (VirtualBox on Windows 10)
- Python 2.7.15rc1（nfcpyは[Python3非対応](https://github.com/nfcpy/nfcpy/issues/47)のため）
- Sony PaSoRi RC-S380
- Suica

（2019/12/19 追記）未検証ですがnfcpyがv1.0.0でPython3対応したみたいです。[https://github.com/nfcpy/nfcpy/issues/47#issuecomment-499693493](https://github.com/nfcpy/nfcpy/issues/47#issuecomment-499693493)

## セットアップ

まずPaSoRiを接続する。PaSoRiをUSBポートに挿して、VirtualBox仮想マシンの設定からUSB、USB デバイスフィルターに`SONY RC-S380/P`を追加。

```shell
$ lsusb
Bus 001 Device 004: ID 054c:06c3 Sony Corp.
...
```

これでOK。

次に[nfcpy](https://pypi.org/project/nfcpy/)のインストール（※Python2に導入すること、virtualenvを使うといいのでは）。

```shell
pip install nfcpy
```

いくらか準備が必要なので、`python -m nfc`を実行する。

```shell
$ python -m nfc
No handlers could be found for logger "nfc.llcp.sec"
This is the 0.13.5 version of nfcpy run in Python 2.7.15rc1
on Linux-4.15.0-33-generic-x86_64-with-Ubuntu-18.04-bionic
I'm now searching your system for contactless devices
** found usb:054c:06c3 at usb:001:004 but access is denied
-- the device is owned by 'root' but you are 'USERNAME'
-- also members of the 'root' group would be permitted
-- you could use 'sudo' but this is not recommended
-- better assign the device to the 'plugdev' group
   sudo sh -c 'echo SUBSYSTEM==\"usb\", ACTION==\"add\", ATTRS{idVendor}==\"054c\", ATTRS{idProduct}==\"06c3\", GROUP=\"plugdev\" >> /etc/udev/rules.d/nfcdev.rules'
   sudo udevadm control -R # then re-attach device
I'm not trying serial devices because you haven't told me
-- add the option '--search-tty' to have me looking
-- but beware that this may break other serial devs
Sorry, but I couldn't find any contactless device
```

一般ユーザーの場合こんなメッセージが出るので、指示通りデバイスを`plugdev`グループに割り当てる。

```shell
sudo sh -c 'echo SUBSYSTEM==\"usb\", ACTION==\"add\", ATTRS{idVendor}==\"054c\", ATTRS{idProduct}==\"06c3\", GROUP=\"plugdev\" >> /etc/udev/rules.d/nfcdev.rules'
```

`/etc/udev/rules.d/nfcdev.rules`はこうなる。

```
SUBSYSTEM=="usb", ACTION=="add", ATTRS{idVendor}=="054c", ATTRS{idProduct}=="06c3", GROUP="plugdev"
...
```

下のコマンドはリロード、のはずだが、うまく動かないのでPaSoRiを抜き差しする（VirtualBoxのメニューからでも、ホストからでも）。

```shell
sudo udevadm control -R
```

もう一度チェックするとメッセージが変わる。

```shell
$ python -m nfc
No handlers could be found for logger "nfc.llcp.sec"
This is the 0.13.5 version of nfcpy run in Python 2.7.15rc1
on Linux-4.15.0-33-generic-x86_64-with-Ubuntu-18.04-bionic
I'm now searching your system for contactless devices
** found usb:054c:06c3 at usb:001:005 but it's already used
-- scan sysfs entry at '/sys/bus/usb/devices/1-2:1.0/'
-- the device is used by the 'port100' kernel driver
-- this kernel driver belongs to the linux nfc subsystem
-- you can remove it to free the device for this session
   sudo modprobe -r port100
-- and blacklist the driver to prevent loading next time
   sudo sh -c 'echo blacklist port100 >> /etc/modprobe.d/blacklist-nfc.conf'
I'm not trying serial devices because you haven't told me
-- add the option '--search-tty' to have me looking
-- but beware that this may break other serial devs
Sorry, but I couldn't find any contactless device
```

```shell
sudo modprobe -r port100
```

でport100なるドライバが止まり、動くようになる。

```shell
sudo sh -c 'echo blacklist port100 >> /etc/modprobe.d/blacklist-nfc.conf'
```

これで次回起動以降、port100が起動しなくなるのかな。

port100を止めたらもう一度チェック。

```shell
$ python -m nfc
No handlers could be found for logger "nfc.llcp.sec"
This is the 0.13.5 version of nfcpy run in Python 2.7.15rc1
on Linux-4.15.0-33-generic-x86_64-with-Ubuntu-18.04-bionic
I'm now searching your system for contactless devices
** found SONY RC-S380/P NFC Port-100 v1.11 at usb:001:005
I'm not trying serial devices because you haven't told me
-- add the option '--search-tty' to have me looking
-- but beware that this may break other serial devs
```

ちゃんと見つかったらOK。

## IDmを読む

短いサンプルコード。

### nf.py

```python
import nfc
from nfc.clf import RemoteTarget

clf = nfc.ContactlessFrontend('usb')
print(clf)

tag = clf.connect(rdwr={
    'on-connect': lambda tag: False
})

print(tag)
```

ICカードがない場合、以下の出力の後（`clf.connect`で）ブロックされる。

```shell
$ python nf.py
No handlers could be found for logger "nfc.llcp.sec"
SONY RC-S380/P on usb:001:005
```

ここでICカードを近づけると、タグが出力（`print(tag)`）される。

```shell
$ python nf.py
No handlers could be found for logger "nfc.llcp.sec"
SONY RC-S380/P on usb:001:005
Type3Tag ID=$$$$$$$$$$$$$$$$ PMM=$$$$$$$$$$$$$$$$ SYS=0003
```

```python
>>> print(type(tag))
<class 'nfc.tag.tt3.Type3Tag'>
>>> 
>>> print(vars(tag)).keys())
['_clf', '_target', 'sys', 'idm', '_ndef', 'pmm', '_nfcid', '_authenticated']
```

`tag.idm`を使えばIDmが取れそうなのでOK。

## 参考

- [Getting started — nfcpy 1.0.3 documentation](https://nfcpy.readthedocs.io/en/latest/topics/get-started.html#read-and-write-tags)
- [交通系電子マネーカードを 黒PaSori RC-S380 + Raspberry Pi + python で - Qiita](https://qiita.com/xshell/items/55302a588b5927dde6b6)
- [RaspberryPiで！SONYのPaSoRi（RC-S380）で（NFC）Felica情報を読み取る！ - KOKENSHAの技術ブログ](https://kokensha.xyz/raspberry-pi/raspberrypi-sony-pasori-rc-s380-read-nfc-felica/)

---

- [Raspberry Pi に nfcpy をインストールする手順 - 2016年3月 私家版 : @jsakamoto](https://devadjust.exblog.jp/23018234/)
- [PaSoRiを使ってpythonでNFCタグを読み書きする - Qiita](https://qiita.com/alt-core/items/abc83b3c1e2dd176717f)
- [nfcpy/tagtool.py at master · nfcpy/nfcpy · GitHub](https://github.com/nfcpy/nfcpy/blob/master/examples/tagtool.py)

---

- [nfc.tag — nfcpy 1.0.3 documentation](https://nfcpy.readthedocs.io/en/latest/modules/tag.html#module-nfc.tag.tt3)
- [http://raspberry.mcoapps.com/archives/128](http://raspberry.mcoapps.com/archives/128)

---

- [[PASMO] FeliCa から情報を吸い出してみる - FeliCaの仕様編 [Android][Kotlin] - Qiita](https://qiita.com/YasuakiNakazawa/items/3109df682af2a7032f8d)
