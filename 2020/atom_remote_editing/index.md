---
# moved from https://aoirint.hatenablog.com/entry/2020/03/19/151704
title: Remote editing with Atom (rmate, sshfs)
date: '2020-03-19 15:17:04'
draft: false
channel: 技術ノート
category: Remote
tags:
- Remote
---
# Remote editing with Atom (rmate, sshfs)

### rmate

- [AtomでSSH越しのファイルを編集する - Qiita](https://qiita.com/informationsea/items/d2576b1ab678965d9a02)

---

1. Install rmate in remote.
2. Install `remote-atom` as Atom package in local.
3. Configure "remote forwarding" of rmate port in local (local:port -> remote:port).
4. Open Atom in local.
5. Execute `rmate file.txt` in remote (ssh). Atom in local will show the file, and you can edit it.

`rmate` cannot be used for Atom Tree View (Project View). But `tree` command in ssh is useful for viewing project file entries.

```sh
tree
tree -L 1
tree -L 1 -A
```

- [【 tree 】コマンド――ディレクトリをツリー状に表示する：Linux基本コマンドTips（179） - ＠IT](https://www.atmarkit.co.jp/ait/articles/1802/01/news025.html)

`autossh -M 0 REMOTE` is also useful.

---

### sshfs

```sh
# Mac
brew cask install osxfuse
brew install sshfs

# Ubuntu
sudo apt install sshfs
```

```sh
sshfs REMOTE: MOUNT_POINT -o follow_symlinks
```

```sh
# Mac
diskutil unmount MOUNT_POINT

# Ubuntu
fusermount -u MOUNT_POINT
```

- [Macでsshfsを使ってローカルからリモートのファイルを触る - Qiita](https://qiita.com/ysk24ok/items/bb148530a55a4e55d99b)
- [command line - How to umount non-sudo sshfs created directory - Ask Ubuntu](https://askubuntu.com/questions/1046816/how-to-umount-non-sudo-sshfs-created-directory)
