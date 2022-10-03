---
title: 'Chrome Remote Desktop （Ubuntu）'
date: '2021-11-13 17:40:00'
updated: '2022-10-03 21:07:00'
draft: false
channel: 技術ノート
category: Network
tags:
  - リモートデスクトップ
---
# Chrome Remote Desktop （Ubuntu）

- <https://remotedesktop.google.com/access>
- <https://qiita.com/ninose14/items/473369d76814174dd58f>

## インストール

- <https://remotedesktop.google.com/headless>

```shell
rm -f ~/.chrome-remote-desktop-session
```

## スクリプトの改変

```shell
cd /opt/google/chrome-remote-desktop

cp chrome-remote-desktop chrome-remote-desktop.bak
```

### /opt/google/chrome-remote-desktop/chrome-remote-desktop

- chrome-remote-desktop 106.0.5249.37

```python
#FIRST_X_DISPLAY_NUMBER = 20
FIRST_X_DISPLAY_NUMBER = 0
```

※ 0の部分には、デスクトップ環境でターミナルを開き、`echo $DISPLAY`で表示される値を入れる。

```python
def launch_session(self, server_args, backoff_time):
  """Launches process required for session and records the backoff time
  for inhibitors so that process restarts are not attempted again until
  that time has passed."""
  logging.info("Setting up and launching session")
  self._init_child_env()
  self.setup_audio()
  self._setup_gnubby()
  #self._launch_server(server_args)
  #if not self._launch_pre_session():
  #  # If there was no pre-session script, launch the session immediately.
  #  self.launch_desktop_session()
  display = self.get_unused_display_number()
  self.child_env['DISPLAY'] = f':{display}'
  self.server_inhibitor.record_started(MINIMUM_PROCESS_LIFETIME,
                                    backoff_time)
  self.session_inhibitor.record_started(MINIMUM_PROCESS_LIFETIME,
                                   backoff_time)
```

```shell
sudo systemctl restart chrome-remote-desktop@${USER}.service
```
