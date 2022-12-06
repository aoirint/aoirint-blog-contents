---
# moved from https://aoirint.hatenablog.com/entry/2018/08/29/034708
title: 'プライベートリポジトリに対してgit cloneがNot Found吐くとき（複数アカウント運用）'
date: '2018-08-29T03:47:08+09:00'
draft: false
channel: 技術ノート
category: Windows
tags:
  - 'Windows'
  - 'Git'
---
# プライベートリポジトリに対してgit cloneがNot Found吐くとき（複数アカウント運用）

## 環境

Windows 10

## 原因

Windows 資格情報に対象のリポジトリのあるアカウント以外の認証情報が記録されていた。

## 解決

[資格情報マネージャー（Credential Manager）](https://support.microsoft.com/ja-jp/help/4026814/windows-accessing-credential-manager)を開いてWindows 資格情報、汎用資格情報からgitを削除。

これでclone時にログイン用のブラウザが表示される（GitHub）。

## 参考
- [GitHubでクローンしてRepository not foundになった | ねこブログ](https://nekosoftware.wordpress.com/2017/02/19/github%E3%81%A7%E3%82%AF%E3%83%AD%E3%83%BC%E3%83%B3%E3%81%97%E3%81%A6repository-not-found%E3%81%AB%E3%81%AA%E3%81%A3%E3%81%9F/)
- [Solved: git clone is not working for a private repo - GitHub Community Forum](https://github.community/t5/How-to-use-Git-and-GitHub/git-clone-is-not-working-for-a-private-repo/m-p/2513#M810)
