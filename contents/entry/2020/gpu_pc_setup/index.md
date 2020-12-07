---
canonical_url: ./
title: GPU PC (Desktop) のセットアップメモ
# og_image:
# twitter_card: summary_large_image
og_description: GPU PC (Desktop) をセットアップする
date: '2020-09-18 20:00:00'
draft: false
category: PC セットアップ
tags:
  - 'PC Setup'
---
# GPU PC (Desktop) のセットアップメモ

[Ubuntu 18.04.5 LTS (Bionic Beaver)](https://releases.ubuntu.com/18.04.5/ "Ubuntu 18.04.5 LTS (Bionic Beaver)")

Ubuntu 18.04、Windows 10のデュアルブート環境を構築する。
プロキシ下を想定しているので、不要な場合は適宜調整。

## 時刻ずれの解消
[Linux環境設定/デュアルブートのWindows時刻をUTCにする - Linuxと過ごす](https://linux.just4fun.biz/?Linux%E7%92%B0%E5%A2%83%E8%A8%AD%E5%AE%9A/%E3%83%87%E3%83%A5%E3%82%A2%E3%83%AB%E3%83%96%E3%83%BC%E3%83%88%E3%81%AEWindows%E6%99%82%E5%88%BB%E3%82%92UTC%E3%81%AB%E3%81%99%E3%82%8B "Linux環境設定/デュアルブートのWindows時刻をUTCにする - Linuxと過ごす")

デュアルブート環境では、BIOSの時刻を介してWindows側の時刻管理（ローカル時間）とUbuntu側の時刻管理（UTC時間）が衝突して、タイムゾーンの時差分、時計がずれてしまう。ここでは、Windows側の時刻管理をUTC時間にするようにレジストリを変更して対処する。

Windows上で管理者権限でコマンドプロンプトを起動し、以下のコマンドを実行する。

```
#!cmd
reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_DWORD /f
```

## NTPサーバの設定（プロキシ環境）
- [Ubuntu 18.04 / 16.04 で時刻合わせの設定を変更する - Sickly Life Blog](https://sicklylife.hatenablog.com/entry/2016/02/03/201557 "Ubuntu 18.04 / 16.04 で時刻合わせの設定を変更する - Sickly Life Blog")
- [systemdのSNTPクライアント機能を使ってみる | Keep the Light Alive](https://yyatsuo.com/post-1297/ "systemdのSNTPクライアント機能を使ってみる | Keep the Light Alive")
- [【Windows】「ハードウェアクロック」と「システムクロック」と「NTP 時刻同期」の関係 | 100%レンタルサーバーを使いこなすサイト](https://go-journey.club/archives/9491 "【Windows】「ハードウェアクロック」と「システムクロック」と「NTP 時刻同期」の関係 | 100%レンタルサーバーを使いこなすサイト")

時刻同期のためプロキシ内のNTPサーバを設定する。

Settings > Details > Date & Timeで使われるNTPサービスは`systemd-timesyncd`。

設定ファイルの`/etc/systemd/timesyncd.conf`を開いて編集する。

```
#!systemd
[Time]
NTP=YOUR_NTP_SERVER
```

```
#!bash
sudo systemctl restart systemd-timesyncd
```

これで時刻が同期される。


## プロキシの設定 (1) Environment Variable, apt
Settings > Networkからプロキシを設定する。設定した内容は`HTTP_PROXY`などの環境変数に出力される。スキーム部分は不要（OS側で付けるので二重プロトコルになってしまう）。

また/etc/apt/apt.confを作成し、プロキシ設定を追記する
```
#!apt
Acquire::http::proxy "http://YOUR_HTTP_PROXY";
Acquire::https::proxy "http://YOUR_HTTP_PROXY";
```

## Package Indexの更新とPackageの更新
```
#!bash
sudo apt update
sudo apt upgrade -y
```

## git、ビルドツールほかのインストール
```
#!bash
sudo apt install -y \
  git \
  build-essential \
  make \
  cmake \
  vim
```

```
#!bash
git config --global core.editor vim
git config --global credential.helper cache
git config --global user.name YOUR_NAME
git config --global user.email YOUR_EMAIL
```

~/.vimrc
```
#!vim
set shiftwidth=4
set tabstop=4
set expandtab
"set number
set cursorline
set smartindent
set laststatus=2
syntax enable

set hlsearch
nmap <Esc><Esc> :nohlsearch<CR><Esc>
```

## 日本語IME（ibus-mozc）、言語サポートのインストール
```
#!bash
sudo apt install -y ibus-mozc

ibus restart
gsettings set org.gnome.desktop.input-sources sources "[('ibus', 'mozc-jp'), ('xkb', 'jp')]"
```

```
#!bash
sudo apt install -y $(check-language-support)
```

## プロキシの設定 (2) git
```
#!bash
git config --global http.proxy ${HTTP_PROXY}
git config --global https.proxy ${HTTPS_PROXY}
```

## pyenv、Pythonのインストール
[pyenv/pyenv: Simple Python version management](https://github.com/pyenv/pyenv#basic-github-checkout)

新しいPythonを使うため、それからシステムと開発環境を分離してモジュール管理をaptとpipで分けて衝突事故を起こさないようにするため、pyenvを使うのが安定。

```
#!bash
git clone https://github.com/pyenv/pyenv.git ~/.pyenv

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc

echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bashrc

source ~/.bashrc
```

- [Common build problems · pyenv/pyenv Wiki](https://github.com/pyenv/pyenv/wiki/common-build-problems "Common build problems · pyenv/pyenv Wiki")

```
#!bash
sudo apt install -y \
  libssl-dev \
  zlib1g-dev \
  libbz2-dev \
  libreadline-dev \
  libsqlite3-dev \
  wget \
  curl \
  llvm \
  libncurses5-dev \
  libncursesw5-dev \
  xz-utils \
  tk-dev \
  libffi-dev \
  liblzma-dev \
  python-openssl

pyenv install 3.8.5
pyenv global 3.8.5
```

## Docker & Docker Composeのインストール
[Install Docker Engine on Ubuntu | Docker Documentation](https://docs.docker.com/engine/install/ubuntu/ "Install Docker Engine on Ubuntu | Docker Documentation")

```
#!bash
sudo apt remove docker docker-engine docker.io containerd runc
sudo apt update
sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -x "${HTTP_PROXY}" \
  | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt update

sudo apt install -y docker-ce docker-ce-cli containerd.io
```

[Install Docker Compose | Docker Documentation](https://docs.docker.com/compose/install/ "Install Docker Compose | Docker Documentation")

```
#!bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.3/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose \
  -x "${HTTP_PROXY}"
sudo chmod +x /usr/local/bin/docker-compose
```

一応、sudoせずにdockerを実行できるようにしておく。

```
#!bash
sudo groupadd docker
sudo adduser $USER docker
```

## プロキシの設定 (3) Docker

```
#!bash
sudo systemctl edit docker
```

このコマンドで開いたファイルに以下を追記する。

```
#!systemd
[Service]
Environment="HTTP_PROXY=YOUR_HTTP_PROXY"
Environment="HTTPS_PROXY=YOUR_HTTP_PROXY"
```

```
#!bash
sudo systemctl restart docker

docker pull hello-world
docker run --rm hello-world
```

正常に実行されればOK。

## CUDA & NVIDIA Driverのインストール
[Start Locally | PyTorch](https://pytorch.org/get-started/locally/ "Start Locally | PyTorch")

PyTorchの対応バージョンを見ながらCUDA Toolkitをインストールする。

[CUDA Toolkit 10.2 Download | NVIDIA Developer](https://developer.nvidia.com/cuda-10.2-download-archive "CUDA Toolkit 10.2 Download | NVIDIA Developer")

CUDA Installerのrunfileを使って、runfileに同梱されているNVIDIA Driverを同時に入れるのが安定（2.5GBくらいで結構重いが）。

[Download runfile installer for Ubuntu 18.04](https://developer.nvidia.com/cuda-10.2-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=runfilelocal)

CUIモードでないとインストールに失敗するので、ダウンロードが終わったら一度デスクトップからログアウトし、Ctrl+Alt+F2などでCUIモードにする。

```
#!bash
sudo systemctl stop gdm
# sudo systemctl stop lightdm # for Ubuntu 16.04
```

これでデスクトップマネージャを止めてからインストールを始める（再起動してGUIモードでログインせずにCUIモードにして実行でもいけるかも）。

```
#!bash
wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda_10.2.89_440.33.01_linux.run
sudo sh cuda_10.2.89_440.33.01_linux.run
```

~/.bashrcに以下を追記する。

```
#!bash
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

PyTorchが入ったあと（どちらを先にいれてもいい）、
```
#!python
import torch
torch.cuda.is_available()
```
でTrueが返ってくればOK。

## OpenCV-Python, Numpy, PyTorchほかのインストール
```
#!bash
pip3 install -U pip
pip3 install -U \
  opencv-python \
  numpy \
  Pillow \
  --proxy "${HTTP_PROXY}"
```

```
#!bash
pip3 install -U \
  torch \
  torchvision \
  torchtext \
  torchaudio \
  --proxy "${HTTP_PROXY}"
```

```
#!bash
pip3 install -U \
  matplotlib \
  tqdm \
  jupyter \
  PyYAML \
  tensorboardX \
  --proxy "${HTTP_PROXY}"
```

## SSH Serverほかのインストール

```
#!bash
sudo apt install -y openssh-server net-tools
```

```
#!bash
ifconfig
```

```
#!bash
sudo vim /etc/ssh/sshd_config
```

必要に応じてパスワード認証を無効にする。

```
#!sshd
PasswordAuthentication no
```

## ownCloudのインストール
[Install package isv:ownCloud:desktop / owncloud-client](https://software.opensuse.org/download/package?project=isv:ownCloud:desktop&package=owncloud-client "Install package isv:ownCloud:desktop / owncloud-client")

```
#!bash
echo 'deb http://download.opensuse.org/repositories/isv:/ownCloud:/desktop/Ubuntu_18.04/ /' | sudo tee /etc/apt/sources.list.d/isv:ownCloud:desktop.list
curl -fsSL https://download.opensuse.org/repositories/isv:ownCloud:desktop/Ubuntu_18.04/Release.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/isv:ownCloud:desktop.gpg > /dev/null
sudo apt update
sudo apt install owncloud-client
```

## その他のインストール

```
#!bash
ssh-keygen -f KEYNAME
```

```
#!bash
sudo apt install tmux autossh
```

## おまけ
### sshのプロキシ設定
- [Mac OSXでHTTP Proxy経由でSSH - Qiita](https://qiita.com/yuyhiraka/items/30766c69fb605fc5f182 "Mac OSXでHTTP Proxy経由でSSH - Qiita")
- [【GitHub】Windows10+プロキシ環境でSSH接続を試みた話 - Qiita](https://qiita.com/_N4H4/items/d48ba7c728f2e1ebd159 "【GitHub】Windows10+プロキシ環境でSSH接続を試みた話 - Qiita")
- [gist:rurban/connect.c](https://gist.github.com/rurban/360940 "connect.c")
    - [ミラー: connect.c](connect.c)

~/.ssh/configの`Host`の下に書く。Windowsの場合は`connect`をビルドする必要がある。

```
#!ssh
    # Linux
    ProxyCommand nc -X connect -x YOUR_HTTP_PROXY %h %p

    # Windows
    ProxyCommand MY_APP_DIRECTORY\connect\connect.exe -H YOUR_HTTP_PROXY %h %p

    # Mac
    ProxyCommand ncat --proxy-type http --proxy YOUR_HTTP_PROXY %h %p
```


```
#!cmd
# for build `connect.c` on Windows
gcc -o connect -lwsock32 connect.c -lws2_32
```
