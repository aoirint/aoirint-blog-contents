---
canonical_url: ./
title: Ubuntu 18.04のKernelをアップデートした（HWE Kernel） + e1000eのDKMS設定
# og_image:
# twitter_card: summary_large_image
og_description: UbuntuのKernelをアップデートしたメモ
date: '2020-09-21 00:00:00'
draft: false
category: Ubuntu
tags:
  - Ubuntu
  - Linux Kernel
  - e1000e
  - DKMS
---
# Ubuntu 18.04のKernelをアップデートした（HWE Kernel） + e1000eのDKMS設定
環境の整理を兼ねて、UbuntuのKernelのアップデートをしたので、そのときのメモ。

## Ubuntu 18.04のKernelをアップデートした（HWE Kernel）
### カーネルバージョンについて
`/lib/modules`を見る限りインストール時のバージョンは`4.10.0-28`で、
`/usr/src`を見る限り`4.15.0-115`をしばらく使ったあと、
`4.16.18`に更新していた。

バージョン`4.10.x`はおそらくUbuntu 16.04をクリーンインストールしたときのもので、
バージョン`4.15.x`は`dist-upgrade`でUbuntu 18.04にアップデートしたときに変更されたと思われる。
その後ソフトウェア導入のためのバージョン合わせかなにかで`4.16.x`にして、そのまま使っていた。

4.15.xから4.16.xにアップデートするときにはUKUU（Ubuntu Kernel Update Utility）を使っていた。
このとき参考にしたサイト： [Upgrade Kernel on Ubuntu 18.04 – Linux Hint](https://linuxhint.com/upgrade-kernel-ubuntu-1804/ "Upgrade Kernel on Ubuntu 18.04 – Linux Hint")

ところで、カーネルバージョンの後ろに付いているハイフン以降の数字はUbuntu Release ABIというらしいのだが、
UKUUを使ってカーネルをインストールするとこの部分がバージョン番号（ハイフンの前）を6ケタの数字に直したようなものになるので、
これはABIとは違いそうだ（ABIは0から255までの範囲のように思われる）。
ABIのドキュメントらしきものがあったので、機会があれば読みたい：[KernelTeam/BuildSystem/ABI - Ubuntu Wiki](https://wiki.ubuntu.com/KernelTeam/BuildSystem/ABI "KernelTeam/BuildSystem/ABI - Ubuntu Wiki")


### UKUUとセキュリティアップデートについて
- [Ubuntu 20.04 その164 - Linux kernelにDoSや任意コード実行の脆弱性・アップデートを - kledgeb](https://kledgeb.blogspot.com/2020/09/ubuntu-2004-164-linux-kerneldos.html "Ubuntu 20.04 その164 - Linux kernelにDoSや任意コード実行の脆弱性・アップデートを - kledgeb")
- [USN-4489-1: Linux kernel vulnerability | Ubuntu security notices | Ubuntu](https://ubuntu.com/security/notices/USN-4489-1 "USN-4489-1: Linux kernel vulnerability | Ubuntu security notices | Ubuntu")

通常の方法でインストールされるカーネルを使っている場合、
Linux Kernelに脆弱性が発見されてもこのようにUbuntu側からセキュリティアップデートが提供される。

しかしUKUUを使って（起動時にデフォルトで使用する）カーネルのバージョンを変更した場合、
これは（デバッグ目的などで）カーネルのバージョンを固定しているのに近いと思われるので、
セキュリティアップデートを手動で行う必要があるのではないかという懸念があった。
実際カーネルバージョンは4.16.xインストール時から固定されていたので、
e1000eの自動ビルドはおそらく無駄で（4.10から4.15では意味があったが）、
4.16.xカーネルのセキュリティアップデートも行われていなかったのではないかと思っている。

### HWEカーネルのインストール
はじめはUKUUを使って5.4.xにアップデートしたものの、
前項の懸念からUbuntuが公式に提供しているHWEカーネルというものを使うことにした。
これならばaptでカーネルが管理され、自動でパッチが適用されるものと思われる。

- [Ubuntuのベースバージョンを変えずにLinuxカーネルをアップグレードする方法 - iberianpigsty](https://iberianpig.github.io/posts/2017-02-06_how_to_upgrade_kernel/ "Ubuntuのベースバージョンを変えずにLinuxカーネルをアップグレードする方法 - iberianpigsty")
- [第278回　Ubuntuカーネルとの付き合い方：Ubuntu Weekly Recipe｜gihyo.jp … 技術評論社](https://gihyo.jp/admin/serial/01/ubuntu-recipe/0278 "第278回　Ubuntuカーネルとの付き合い方：Ubuntu Weekly Recipe｜gihyo.jp … 技術評論社")

Ubuntu 18.04の場合、`linux-generic-hwe-18.04`が安定版、`linux-generic-hwe-18.04-edge`が開発版ということだろうか。
今回は安定性を重視するので安定版を選んでアップデートする。

```
sudo apt install linux-generic-hwe-18.04
```

このあとから以下のようになったので、このシステムでaptが管理していたカーネルバージョンは`4.15.0-115`だったことがわかる。
```
The following packages were automatically installed and are no longer required:
  linux-headers-4.15.0-115 linux-headers-4.15.0-115-generic
  linux-image-4.15.0-115-generic linux-modules-4.15.0-115-generic
  linux-modules-extra-4.15.0-115-generic
```

### 余談：UKUUを使ったカーネル導入とデフォルトカーネルとgrubの設定
ここで、grubのデフォルトエントリがUKUUで入れた5.4.xカーネルのままで、
新たにインストールしたUbuntu HWEカーネルではなかった。
この項ではこの理由を検討するが、実際にはgrub（またはUbuntuに同梱されているgrub）のデフォルトの挙動であったので、
余談である。

（カーネルインストール時に自動で呼ばれるが）`update-grub`実行時には以下のように表示される。

```sh
$ sudo update-grub
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.4.42-050442-generic      <-- UKUUで入れたカーネル（デフォルト）
Found initrd image: /boot/initrd.img-5.4.42-050442-generic
Found linux image: /boot/vmlinuz-5.4.0-47-generic           <-- 新しいカーネル（HWE）
Found initrd image: /boot/initrd.img-5.4.0-47-generic
Found linux image: /boot/vmlinuz-4.16.18-041618-generic     <-- 現在のカーネル（UKUU）
Found initrd image: /boot/initrd.img-4.16.18-041618-generic
Found linux image: /boot/vmlinuz-4.15.0-117-generic         <-- 未使用?
Found initrd image: /boot/initrd.img-4.15.0-117-generic
Found linux image: /boot/vmlinuz-4.15.0-115-generic         <-- aptが管理しているカーネル
Found initrd image: /boot/initrd.img-4.15.0-115-generic
Found Windows Boot Manager on /dev/sdc1@/EFI/Microsoft/Boot/bootmgfw.efi
Adding boot menu entry for EFI firmware configuration
done
```

`/boot/grub/grub.cfg`をみると、
```grub
menuentry 'Ubuntu' --class ubuntu --class gnu-linux --class gnu --class os $menu
entry_id_option 'gnulinux-simple-ed8cbed5-714e-4201-b606-c41d570f834d' {
        recordfail
        load_video
        gfxmode $linux_gfx_mode
        insmod gzio
        if [ x$grub_platform = xxen ]; then insmod xzio; insmod lzopio; fi
        insmod part_gpt
        insmod ext2
        set root='hd0,gpt1'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,gpt1 --hint-ef
i=hd0,gpt1 --hint-baremetal=ahci0,gpt1  ed8cbed5-714e-4201-b606-c41d570f834d
        else
          search --no-floppy --fs-uuid --set=root ed8cbed5-714e-4201-b606-c41d570f834d
        fi
        linux   /boot/vmlinuz-5.4.42-050442-generic root=UUID=ed8cbed5-714e-4201-b606-c41d570f834d ro  quiet splash $vt_handoff
        initrd  /boot/initrd.img-5.4.42-050442-generic
}
```

このような記述があったので、grubメニューの0番目に表示される`Ubuntu`という項目は`5.4.42-050442`のカーネルを起動するようになっていることがわかる。
この設定は`apt install linux-generic-hwe-18.04`のあと`update-grub2`しても変わらなかった。
おそらくUKUUが自動で設定したと思われる（以下で調べるが、実際には違った）が、これをaptの管理するカーネルになるようにしたい。

```grub
### BEGIN /etc/grub.d/10_linux ###
```

とあるので、この部分は`/etc/grub.d/10_linux`からインクルードされている。

grub.d以下は標準出力をgrub.cfgに書き出し、エラー出力を`update-grub`したときに表示するような
シェルスクリプトになっているようで、自動的にカーネルイメージを見つける作りになっているようだ。

grubのメニューには`Ubuntu`（menuentry）、`Advanced options for Ubuntu`（submenu./men）のように並ぶ。

[【 grub2-set-default／grub-set-default 】コマンド――GRUB 2のデフォルト起動メニューを設定する：Linux基本コマンドTips（277） - ＠IT](https://www.atmarkit.co.jp/ait/articles/1901/31/news048.html "【 grub2-set-default／grub-set-default 】コマンド――GRUB 2のデフォルト起動メニューを設定する：Linux基本コマンドTips（277） - ＠IT")


一番上のmenuentryを生成している`/etc/grub.d/10_linux`の一部：

```sh
    linux_entry "${OS}" "${version}" simple \
    "${GRUB_CMDLINE_LINUX} ${GRUB_CMDLINE_LINUX_DEFAULT}"
```

関数linux_entryでは、第3引数に`simple`が指定されている場合、以下のようなコード（一部）でmenuentryを生成する。

```sh
linux_entry ()
{
  os="$1"
  version="$2"
  type="$3"
  args="$4"

（略）

      echo "menuentry '$(echo "$os" | grub_quote)' ${CLASS} \$menuentry_id_option 'gnulinux-simple-$boot_device_id' {" | sed "s/^/$submenu_indentation/"

（略）

        sed "s/^/$submenu_indentation/" << EOF
        linux   ${rel_dirname}/${basename} root=${linux_root_device_thisversion} ro ${args}
EOF
```

${rel_dirname}や${basename}は関数`version_find_latest $list`から生成しているようだ。
menuentryのループを回している部分では、以下のように`version_find_latest $list`を呼び出していて、
`is_top_level`は初回のループで`true`になり、このとき一番上のメニューを生成する。
関数`linux_entry`を呼び出しているところは上に書いたものと同じ部分である。

```sh
is_top_level=true
while [ "x$list" != "x" ] ; do
  linux=`version_find_latest $list`

（略）

  if [ "x$is_top_level" = xtrue ] && [ "x${GRUB_DISABLE_SUBMENU}" != xy ]; then
    linux_entry "${OS}" "${version}" simple \
    "${GRUB_CMDLINE_LINUX} ${GRUB_CMDLINE_LINUX_DEFAULT}"

    submenu_indentation="$grub_tab"

    if [ -z "$boot_device_id" ]; then
        boot_device_id="$(grub_get_device_id "${GRUB_DEVICE}")"
    fi
    # TRANSLATORS: %s is replaced with an OS name
    echo "submenu '$(gettext_printf "Advanced options for %s" "${OS}" | grub_quote)' \$menuentry_id_option 'gnulinux-advanced-$boot_device_id' {"
    is_top_level=false
  fi

```

`$list`は以下のように生成される。

```
        list=
        for i in /boot/vmlinuz-* /vmlinuz-* /boot/kernel-* ; do
            if grub_file_is_not_garbage "$i" ; then list="$list $i" ; fi
        done ;;
```

これで問題は関数`version_find_latest`のアルゴリズムということがわかった。

関数`version_find_latest`は`/etc/grub.d/10_linux`にはないので、
おそらくファイル先頭近くにある`. "$pkgdatadir/grub-mkconfig_lib"`の部分で読み出されていると思われる。

`$pkgdatadir`というのがどこかわからなかったので、`find`で雑に検索を掛けたところ、`/usr/lib/grub/grub-mkconfig_lib`を読み出していそうなことがわかった。
以下は`/usr/lib/grub/grub-mkconfig_lib`の一部である。確かに`version_find_latest`があった。

```sh
version_test_gt ()
{
  version_test_gt_sedexp="s/[^-]*-//;s/[._-]\(pre\|rc\|test\|git\|old\|trunk\)/~\1/g"
  version_test_gt_a="`echo "$1" | sed -e "$version_test_gt_sedexp"`"
  version_test_gt_b="`echo "$2" | sed -e "$version_test_gt_sedexp"`"
  version_test_gt_cmp=gt
  if [ "x$version_test_gt_b" = "x" ] ; then
    return 0
  fi

  # GRUB_FLAVOUR_ORDER is an ordered list of kernels, in decreasing
  # priority. Any items in the list take precedence over other kernels,
  # and earlier flavours are preferred over later ones.
  for flavour in ${GRUB_FLAVOUR_ORDER:-}; do
    version_test_gt_a_preferred=$(echo "$version_test_gt_a" | grep --  "-[0-9]*-$flavour\$")
    version_test_gt_b_preferred=$(echo "$version_test_gt_b" | grep --  "-[0-9]*-$flavour\$")

    if [ -n "$version_test_gt_a_preferred" -a -z "$version_test_gt_b_preferred" ] ; then
      return 0
    elif [ -z "$version_test_gt_a_preferred" -a -n "$version_test_gt_b_preferred" ] ; then
      return 1
    fi
  done

  case "$version_test_gt_a:$version_test_gt_b" in
    *.old:*.old) ;;
    *.old:*) version_test_gt_a="`echo "$version_test_gt_a" | sed -e 's/\.old$//'`" ; version_test_gt_cmp=gt ;;
    *:*.old) version_test_gt_b="`echo "$version_test_gt_b" | sed -e 's/\.old$//'`" ; version_test_gt_cmp=ge ;;
  esac
  dpkg --compare-versions "$version_test_gt_a" "$version_test_gt_cmp" "$version_test_gt_b"
  return "$?"
}

version_find_latest ()
{
  version_find_latest_a=""
  for i in "$@" ; do
    if version_test_gt "$i" "$version_find_latest_a" ; then
      version_find_latest_a="$i"
    fi
  done
  echo "$version_find_latest_a"
}
```

`GRUB_FLAVOUR_ORDER`がおそらく`/etc/default/grub`で指定されていなければ、
`dpkg --compare-versions PKG_A COMPARATOR PKG_B`によってソートされそうなことがわかった。

ここでUKUUについて調べてみると、
UKUUは2019年1月に有料化しているようだが、
ppa:teejee2008/ppaとコードベースは残っていた。

- [Ukuu v19.01 – TeejeeTech](https://teejeetech.in/2019/01/20/ukuu-v19-01/ "Ukuu v19.01 – TeejeeTech")

> gothicVI
> January 22, 2019 at 1:18 am
>
> So ukuu now completely turned into a closed source project?

> Tony George
> January 22, 2019 at 12:09 pm
>
> Yes.
> Older versions are still open-source. Somebody can develop that version further if they have the time and interest. I may open the source again if I stop working on it (it won’t happen anytime soon).

- [teejee2008/ukuu: A paid version of Ukuu is now available with more features. https://teejeetech.in/2019/01/20/ukuu-v19-01/ Kernel Update Utility for Ubuntu-based distributions. Provides desktop notifications when new mainline kernel is available. Lists kernels from http://kernel.ubuntu.com/~kernel-ppa/mainline/ with options to install and remove.](https://github.com/teejee2008/ukuu "teejee2008/ukuu: A paid version of Ukuu is now available with more features. https://teejeetech.in/2019/01/20/ukuu-v19-01/ Kernel Update Utility for Ubuntu-based distributions. Provides desktop notifications when new mainline kernel is available. Lists kernels from http://kernel.ubuntu.com/~kernel-ppa/mainline/ with options to install and remove.")

grubの設定をいじっているソースコードを探したところ、`update-grub`を呼び出すくらいで特に優先度を設定するようなことはしていなさそうだったので、
`dpkg --compare-versions`を使ったソートの結果、単純に最初に来たカーネルがデフォルト（一番上のmenuentry）に使われていそうとわかった（なにも特殊なことはない普通の動作だ..）。

[ukuu/LinuxKernel.vala#L1298 at master · teejee2008/ukuu](https://github.com/teejee2008/ukuu/blob/master/src/Common/LinuxKernel.vala#L1298 "ukuu/LinuxKernel.vala at master · teejee2008/ukuu")

問題はdpkgがUbuntu HWEカーネルよりUKUUで入れた5.4.xカーネルの方が新しいと判断していることが原因で、
UKUUは特殊なことをしていないとわかったので、
単純にUKUUから入れたカーネルを削除して`update-grub`すればデフォルトが（もっとも新しい）HWEカーネルになりそうだとわかった。
一度別のカーネル（HWEでOK）で起動して、UKUUのGUIを使ってUKUU側の5.4.xを削除（ふつうに選択してRemove）すればデフォルトでもっとも新しいHWEカーネルが起動するようになる。


## e1000eのDKMS設定
### Intel NICのドライバe1000eについて
```sh
$ find /lib/modules/5.4.0-47-generic -name e1000e*
/lib/modules/5.4.0-47-generic/kernel/drivers/net/ethernet/intel/e1000e
/lib/modules/5.4.0-47-generic/kernel/drivers/net/ethernet/intel/e1000e/e1000e.ko
```

このようにデフォルトでe1000eのドライバがカーネルに付属しているようなのだが、
（デバイスによっては?）チェックサム検証に失敗（`The NVM Checksum Is Not Valid` by `netdev.c`）する問題がある。

```sh
# lspci -vvv
00:1f.6 Ethernet controller: Intel Corporation Ethernet Connection (2) I219-V
	Subsystem: Intel Corporation Ethernet Connection (2) I219-V
	Control: I/O- Mem+ BusMaster- SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx-
	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
	Interrupt: pin A routed to IRQ 16
	Region 0: Memory at df100000 (32-bit, non-prefetchable) [size=128K]
	Capabilities: [c8] Power Management version 3
		Flags: PMEClk- DSI+ D1- D2- AuxCurrent=0mA PME(D0+,D1-,D2-,D3hot+,D3cold+)
		Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=1 PME-
	Capabilities: [d0] MSI: Enable- Count=1/1 Maskable- 64bit+
		Address: 00000000fee00338  Data: 0000
	Capabilities: [e0] PCI Advanced Features
		AFCap: TP+ FLR+
		AFCtrl: FLR-
		AFStatus: TP-
	Kernel modules: e1000e  <-- これが動かない
```

エラーログ
```sh
$ zegrep e1000e /var/log/kern.log*
kernel: [    1.296005] e1000e: Intel(R) PRO/1000 Network Driver - 3.2.6-k
kernel: [    1.296006] e1000e: Copyright(c) 1999 - 2015 Intel Corporation.
kernel: [    1.296023] e1000e 0000:00:1f.6: enabling device (0000 -> 0002)
kernel: [    1.296199] e1000e 0000:00:1f.6: Interrupt Throttling Rate (ints/sec) set to dynamic conservative mode
kernel: [    1.546779] e1000e 0000:00:1f.6: The NVM Checksum Is Not Valid
kernel: [    1.588850] e1000e: probe of 0000:00:1f.6 failed with error -5
```

そのため結局は無効にして自分でビルドする必要がある。

[ダウンロード Linux * での PCIe * Intel®ギガビット・イーサネット・ネットワーク接続向けインテル®ネットワーク・アダプター・ドライバー](https://downloadcenter.intel.com/ja/download/15817 "ダウンロード Linux * での PCIe * Intel®ギガビット・イーサネット・ネットワーク接続向けインテル®ネットワーク・アダプター・ドライバー")

解凍したあと、`src/nvm.c`の`e1000e_validate_nvm_checksum_generic`が0を返すように編集する。

```sh
# チェックサム検証のスキップ
sed -i "/s32 e1000e_validate_nvm_checksum_generic(struct e1000_hw \*hw)/N;s/\n{/\n{return 0;/" nvm.c
```

### UKUUとe1000eのビルドについて
今回はUKUUを使わないためこれは余談なのだが、UKUUで導入したカーネルでe1000eをビルドするときには、チェックサム検証の問題に加えてABIに関連した問題が起こる。
ここにe1000eのソースコード（`kcompat.h`）の一部を引用するが、以下のようにe1000eのプログラム内でABIのチェックが行われていて、
4.16.xのカーネルをUKUUで導入した際はこのバージョンチェックをコメントアウトする必要があった。

```c
/* Ubuntu Release ABI is the 4th digit of their kernel version. You can find
 * it in /usr/src/linux/$(uname -r)/include/generated/utsrelease.h for new
 * enough versions of Ubuntu. Otherwise you can simply see it in the output of
 * uname as the 4th digit of the kernel. The UTS_UBUNTU_RELEASE_ABI is not in
 * the linux-source package, but in the linux-headers package. It begins to
 * appear in later releases of 14.04 and 14.10.
 *
 * Ex:
 * <Ubuntu 14.04.1>
 *  $uname -r
 *  3.13.0-45-generic
 * ABI is 45
 *
 * <Ubuntu 14.10>
 *  $uname -r
 *  3.16.0-23-generic
 * ABI is 23
 */

(略)

#if UTS_UBUNTU_RELEASE_ABI > 255
#error UTS_UBUNTU_RELEASE_ABI is too large...
#endif /* UTS_UBUNTU_RELEASE_ABI > 255 */
```

```sh
# ABIチェックのスキップ
sed -i "s/#error UTS_UBUNTU_RELEASE_ABI is too large.../\/\/#error UTS_UBUNTU_RELEASE_ABI is too large.../" kcompat.h
```

他にe1000e以外で注意が必要かもしれない点として、
おそらくメジャーバージョンが1ケタであるせいで、この数字の始まりが0からになっているため、
これを直接Cコードに埋め込んだりすると8進数扱いされて（さらに数字に8以上が含まれていて）ビルドが通らないということがあった（どのソフトウェアか覚えていないが）。

### これまでのe1000e自動ビルドについて
Linuxにはカーネルバージョンをアップデートしたときにドライバなどのモジュールを再ビルドするための
DKMS（Dynamic Kernel Module Support）というソフトウェアがあるのだが、
導入当時はこれを使うキャパシティがなかったので、当時でもなんとなく使い方のわかっていたsystemdを使って
起動時に毎回e1000eを自動ビルド・再インストールするという荒い方法で継続的に動作させていた。

/etc/systemd/system/uscript-e1000e.service

```systemd
[Unit]
Description=Make Install e1000e

[Service]
Type=oneshot
ExecStart=/etc/uscript/e1000e

TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
```

/etc/uscript/e1000e

```
#!/bin/bash

modprobe -r e1000e
make clean -C /etc/uscript/e1000e-latest/src
make install -C /etc/uscript/e1000e-latest/src
modprobe e1000e
```

カーネルバージョンを更新するにあたって、DKMSに移行することとし、これは不要になったので削除した。

```sh
$ sudo systemctl stop uscript-e1000e.service
$ sudo systemctl disable uscript-e1000e.service
Removed /etc/systemd/system/multi-user.target.wants/uscript-e1000e.service.
$ sudo rm /etc/systemd/system/uscript-e1000e.service
$ sudo rm /etc/uscript/e1000e

$ sudo modprobe -r e1000e
$ sudo make uninstall -C /etc/uscript/e1000e-latest/src
$ sudo rm -r /etc/uscript/e1000e-latest
```

### DKMSを使ったe1000e自動ビルドについて
[Ubuntu 16.04でRTL8189FTV （RTL8188FU）ドライバのDKMS化 (r271-635)](https://netlog.jpn.org/r271-635/2019/06/ubuntu_rtl8189ftv_dkms.html "Ubuntu 16.04でRTL8189FTV （RTL8188FU）ドライバのDKMS化 (r271-635)")

これを参考にカーネルアップデート時に自動でリビルドするDKMSに対応させる作業をした。

まず、あらかじめ`dkms`をインストールしておく。
もし先にカーネルを更新してしまって`dkms`を取得できないときは、一度手動で`e1000e`をビルドすればOK（`make uninstall`を忘れずに）。

```sh
sudo apt install dkms
```

まずは`/usr/src`以下にソースディレクトリをコピーする。
今回の場合、`e1000e-3.8.4.tar.gz`を解凍した`e1000e-3.8.4`ディレクトリを`/usr/src/e1000e-3.8.4`としてコピーする。
そして`/usr/src/e1000e-3.8.4/dkms.conf`を作成する。

dkms.confの細かい説明：[Ubuntu Manpage: dkms - Dynamic Kernel Module Support](https://manpages.ubuntu.com/manpages/bionic/man8/dkms.8.html#dkms.conf "Ubuntu Manpage: dkms - Dynamic Kernel Module Support")

ディレクトリ構造
```
| /usr/src/e1000e-3.8.4/
|-- README
|-- dkms.conf  <-- New!
|-- ...
|-- src/
|----- Makefile
|----- e1000.h
|----- ...
```

dkms.conf
```dkms
PACKAGE_NAME="e1000e"
PACKAGE_VERSION="3.8.4"
CLEAN="cd src; make clean"
BUILT_MODULE_NAME[0]="e1000e"
BUILT_MODULE_LOCATION[0]="src/"
DEST_MODULE_NAME[0]="e1000e-dkms"
MAKE[0]="cd src; make -j$(nproc)"
DEST_MODULE_LOCATION[0]="/updates/dkms"
AUTOINSTALL="yes"
REMAKE_INITRD="yes"
```

これだけでDKMSに登録する準備が完了した。次はDKMSにこのソースディレクトリを登録する。
DKMSはデフォルトで`/usr/src`以下のディレクトリを見に行くように思われる。

```sh
$ sudo dkms add e1000e/3.8.4

Creating symlink /var/lib/dkms/e1000e/3.8.4/source ->
                 /usr/src/e1000e-3.8.4

DKMS: add completed.
```

`/var/lib/dkms/e1000e/3.8.4/source`からのシンボリックリンクが張られ、DKMSに登録された。DKMSから削除するには：

```sh
$ sudo dkms remove e1000e/3.8.4 --all

------------------------------
Deleting module version: 3.8.4
completely from the DKMS tree.
------------------------------
Done.
```

次はビルドしてみる。

```sh
$ sudo dkms build e1000e/3.8.4

Kernel preparation unnecessary for this kernel.  Skipping...

Building module:
cleaning build area...
cd src; make -j8....
Signing module:
 - /var/lib/dkms/e1000e/3.8.4/5.4.0-47-generic/x86_64/module/e1000e-dkms.ko
Secure Boot not enabled on this system.
cleaning build area...

DKMS: build completed.
```

そしてインストール。

```sh
$ sudo dkms install e1000e/3.8.4

e1000e-dkms:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/5.4.0-47-generic/updates/dkms/

depmod...

DKMS: install completed.
```

デフォルトの`e1000e`を無効化する。
```sh
sudo modprobe -r e1000e

# 再起動時にロードされないようにnouveauにならって設定しようとしたが、うまくいかなかった
# printf "# disable default e1000e driver; use self-built version instead.\nblacklist e1000e\n" | sudo tee /etc/modprobe.d/blacklist-e1000e.conf
```

自動でモジュールが読み込まれないと思われるので、`dkms`の方の`e1000e`を`modprobe`を使って手動で読み込む。
```sh
sudo modprobe e1000e-dkms
modinfo e1000e-dkms
```

[オンボードのEthernetコントローラ(I219-V)がUbuntuで動かない時の対処 | Ray's Note](https://blog.spiralray.net/archives/474 "オンボードのEthernetコントローラ(I219-V)がUbuntuで動かない時の対処 | Ray's Note")
