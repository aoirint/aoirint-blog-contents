---
canonical_url: ./
title: GitHubの認証方法をPATにするにあたって
# og_image:
# twitter_card: summary_large_image
og_description: GitHubの認証方法をPATにするにあたって
date: '2020-12-18 11:00:00'
draft: false
category: GitHub
tags:
  - GitHub
  - 認証
---
# GitHubの認証方法をPATにするにあたって

GitHubがパスワードによるGitアクセスを無効化する（正確にはパーソナルアクセストークン認証を必須化する、だが現状SSH認証は残る）旨のアナウンスをした（[Token authentication requirements for Git operations - The GitHub Blog](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/)）。

これまで、コマンドラインGit操作のためのGitHubの認証にはパスワード認証（HTTPS Basic認証）を使っていた。
一応、リポジトリを操作するたびにパスワードを入力するのは面倒なので、
cacheは有効化していた。（`git config --global credential.helper cache`）。

パスワード認証は、持ち出せないPCやVPS、モバイル端末や新しいPCから一時的にリポジトリにアクセスする必要があるときに便利だった。
もちろん、キーロガーやgitコマンドなどシステムがハックされている可能性を考えると望ましくないことはわかっていたが..。
以前はSSH認証を使っていた時期もあったが、ブラウザ上での鍵の登録削除操作が必要でこういった一時的な認証には向かない（この記事ではこの問題は解決しない。都度適切な認証情報を用意するか、平文保存するか、手元で認証情報を持つ必要のない自動デプロイを採用することになるかもしれない）。
また、同じGitホスティングサービスに複数アカウントを作っているとき、SSH認証の場合デバイス数×アカウント数と大量に鍵を管理する必要が出てくる（この記事ではこの問題も解決しない。しかし自分のサブアカウント[@kanomiya](https://github.com/kanomiya)は現在アーカイブ用の非アクティブなので問題ないとする）。

自分の認識として、GitHubのGit操作をする認証にはパスワード認証（HTTPS Basic認証）、SSH認証、パーソナルアクセストークンがある（GitHub ActionsのトークンやOAuthなどは除く）。

SSH認証を使っていなかった理由は先に述べた問題（一時的なアクセス・大量の鍵管理）のほかに、プロキシがある。
自分の環境では、ネットワーク/場所が変わるたびにプロキシ設定を切り替える必要があり、
HTTPS認証の場合、毎回`git config --global http.proxy`あるいは`git config --global --unset http.proxy`を打つことになっている（あるいはシェルスクリプト。`global`なのはリポジトリ別に設定を残さないためと`clone`ができないから）。
ここでSSHを使うとHTTPプロキシ経由で通信するためのProxyCommandを設定することになるが、これがやっかいである。
まず、OSごとにプロキシコマンドが異なる。`connect -H`（Windows）、`ncat --proxy-type http --proxy`（macOS）、`nc -X connect -x`（Linux）とそれぞれのOS用の設定を用意することになる。
また、プロキシのON/OFF自体については、ふつうに`ssh`でつなぐときは、ワイルドカード設定を使って接続先名でプロキシを切り替えるようにしているのだが、
Gitの場合は接続先名を変えるために`git remote set-url`する必要があり、これにはリポジトリのパスを記憶して頻繁に入力する必要が出てきてしまう。今のところそのような認識であるので、パーソナルアクセストークンを使った認証に切り替えていく。

パーソナルアクセストークン（PAT）は、パスワードの代わりに利用可能な認証情報で、権限を絞ったトークンを生成できる。[GitHub Package Registry](https://docs.github.com/en/free-pro-team@latest/packages/learn-github-packages/about-github-packages)を利用するときに使っていて（`docker login docker.pkg.github.com`。パスワード認証不可。現在GitHubのDockerレジストリはpull認証不要の[GitHub Container Registry](https://docs.github.com/en/free-pro-team@latest/packages/guides/about-github-container-registry) `ghcr.io` に置き換えられる予定でパブリックベータ中）、好印象を持っていた（PyPIやDocker Hubでもトークン認証が便利）。また、GitHubはSSH認証よりも[HTTPS認証を推奨している](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/set-up-git#next-steps-authenticating-with-github-from-git)。PATの設定箇所は若干わかりにくいが、Settings > Developer settings > Personal access tokensから設定できる。

主要機の切り替えにあたって、トークンを毎回入力するわけにはいかないので、入力されたパスワード（トークン）を認証情報マネージャを使って永続化する設定をする。Windows（Git for Windows）、macOSでは自動で設定されるように思うが、Ubuntuでは手動設定することになる。

`libgnome-keyring`の`git-credential-gnome-keyring`というのがあるが、`libgnome-keyring`が非推奨になっている（[linux - Error when using Git credential helper with gnome-keyring as Sudo - Stack Overflow](https://stackoverflow.com/a/40312117)、[[libgnome-keyring] Deprecate libgnome-keyring. Use libsecret instead](https://mail.gnome.org/archives/commits-list/2014-January/msg01585.html)）ようなので`libsecret`を使う（裏側では同じ`gnome-keyring`が動いているようだが）。
どちらもほぼ手順は同じだが、開発用パッケージからソースコードを取得したあと、手動で`make`するという謎手順がある（\#パッケージマネージャとは）。これで一応保存時の暗号化（復元可能）は施された状態で永続化できる。


## libsecretの場合
[password - What is the correct way to use git with gnome-keyring and http(s) repos? - Ask Ubuntu](https://askubuntu.com/a/959662)

```sh
sudo apt install libsecret-1-0 libsecret-1-dev
sudo make --directory=/usr/share/doc/git/contrib/credential/libsecret
git config --global credential.helper /usr/share/doc/git/contrib/credential/libsecret/git-credential-libsecret
```


## libgnone-keyringの場合（非推奨）
[zenn:tlls/gnome-keyringでgitクレデンシャルを保存する](https://zenn.dev/tlls/articles/gnome-keyring-git-credential)

```sh
sudo apt install libgnome-keyring-dev
sudo make --directory=/usr/share/doc/git/contrib/credential/gnome-keyring
git config --global credential.helper /usr/share/doc/git/contrib/credential/gnome-keyring/git-credential-gnome-keyring
```
