---
# moved from https://aoirint.hatenablog.com/entry/2020/03/19/063201
title: systemd service
date: '2020-03-19 06:32:01'
draft: false
channel: 技術ノート
category: Ubuntu
tags:
- Ubuntu
- AutoStart
---
# systemd service

テンプレート（一般ユーザ権限、bashrc使用）

```systemd
[Unit]
Description=My Service

[Service]
Type=simple
User=user
Group=user
Restart=always
WorkingDirectory=WORKING_DIR
ExecStart=/bin/bash -c "COMMAND"

[Install]
WantedBy=multi-user.target
```

```sh
sudo ln -s my_service.service /etc/systemd/system/
sudo systemctl enable my_service
sudo systemctl restart my_service

# After you edit service file
sudo systemctl daemon-reload

# Check log
sudo systemctl status my_service
```

If you are using python, maybe you need to set environment `PYTHONUNBUFFERED=1` to see log.

```
# In [Service] Section
Environment="PYTHONUNBUFFERED=1"

Environment="KEY=VALUE"
```

- [Systemdを使ってさくっと自作コマンドをサービス化してみる - Qiita](https://qiita.com/DQNEO/items/0b5d0bc5d3cf407cb7ff)
- [systemd サービスユニット覚書 - Qiita](https://qiita.com/ch7821/items/369090459769c603bb6b)
- [Systemd メモ書き - Qiita](https://qiita.com/a_yasui/items/f2d8b57aa616e523ede4)
- [Systemd Unit File チートシート - Qiita](https://qiita.com/ukiuni@github/items/400e4fdaae14b9bc0fcf)
