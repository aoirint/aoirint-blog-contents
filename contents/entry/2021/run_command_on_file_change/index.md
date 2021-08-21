---
title: ファイル変更時にコマンド実行（watchmedo, watchexec）
date: '2021-08-22 06:30:00'
draft: false
category: Command Utility
tags:
  - Command Utility
  - Filesystem
---

# ファイル変更時にコマンド実行（watchmedo, watchexec）

## watchmedo (watchdog)

- <https://github.com/gorakhargosh/watchdog>

```shell
watchmedo shell-command -R --command 'make build' ./src

# プロセスが動いていても停止して再実行する
watchmedo auto-restart -R -d ./src -- make serve
```

### インストール
```shell
pip3 install watchdog[watchmedo]==2.1.3
```

※ 2.1.4 on Ubuntuはバグがありそう

```
Exception in thread Thread-1:
Traceback (most recent call last):
  File "$HOME/.pyenv/versions/3.9.6/lib/python3.9/threading.py", line 973, in _bootstrap_inner
    self.run()
  File "$HOME/.pyenv/versions/3.9.6/lib/python3.9/site-packages/watchdog/observers/api.py", line 199, in run
    self.dispatch_events(self.event_queue, self.timeout)
  File "$HOME/.pyenv/versions/3.9.6/lib/python3.9/site-packages/watchdog/observers/api.py", line 372, in dispatch_events
    handler.dispatch(event)
  File "$HOME/.pyenv/versions/3.9.6/lib/python3.9/site-packages/watchdog/events.py", line 282, in dispatch
    self.event_dispatch_map[event.event_type](event)
AttributeError: 'RootHandler' object has no attribute 'event_dispatch_map'
```

## watchexec
- <https://github.com/watchexec/watchexec>
- <https://superuser.com/questions/181517/how-to-execute-a-command-whenever-a-file-changes>

```shell
watchexec -w ./src -- make build

# プロセスが動いていても停止して再実行する
watchexec -r -w ./src -- make serve

# 初回実行しない
watchexec -p -w ./src -- make event-trigger
```

### インストール
#### Rustのインストール
- <https://www.rust-lang.org/ja/learn/get-started>

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### watchexecのインストール
```shell
cargo install watchexec-cli
```
