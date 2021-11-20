---
title: 'Chrome Remote Desktop （Ubuntu）'
date: '2021-11-13 17:40:00'
draft: false
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

```python
#FIRST_X_DISPLAY_NUMBER = 20
FIRST_X_DISPLAY_NUMBER = 0
```

```python
def launch_session(self, x_args):
  self._init_child_env()
  self._setup_pulseaudio()
  self._setup_gnubby()
  #self._launch_x_server(x_args)
  #if not self._launch_pre_session():
  #  # If there was no pre-session script, launch the session immediately.
  #  self.launch_x_session()
  display = self.get_unused_display_number()
  self.child_env["DISPLAY"] = f":{display}"
```

```shell
sudo systemctl restart chrome-remote-desktop@${USER}.service
```
