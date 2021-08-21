---
title: ファイル変更時にコマンド実行（watchexec）
date: '2021-08-22 06:30:00'
draft: false
category: Command Utility
tags:
  - Command Utility
  - Rust
  - Ubuntu
---

# ファイル変更時にコマンド実行（watchexec）

- https://superuser.com/questions/181517/how-to-execute-a-command-whenever-a-file-changes

## watchexec
```shell
watchexec -w . -- make build

# プロセスが動いていても停止して再実行する
watchexec -r -w . -- make serve

# 初回実行しない
watchexec -p -w . -- make event-trigger
```

### インストール
#### Rustのインストール
- https://www.rust-lang.org/ja/learn/get-started

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### watchexecのインストール
- https://github.com/watchexec/watchexec

```shell
cargo install watchexec-cli
```
