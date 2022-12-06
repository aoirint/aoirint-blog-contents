---
# moved from https://aoirint.hatenablog.com/entry/2020/03/27/103013
title: nginx basic auth, reverse proxy, static files
date: '2020-03-27T10:30:13+09:00'
draft: false
channel: 技術ノート
category: nginx
tags:
- nginx
- Web
---
# nginx basic auth, reverse proxy, static files

### Basic auth
- [Nginx で Basic 認証 - Qiita](https://qiita.com/kotarella1110/items/be76b17cdbe61ff7b5ca)

```
        auth_basic "Authentication Required";
        auth_basic_user_file DIRECTORY/.htpasswd;
```

### Reverse proxy
- [Nginxによるリバースプロキシの設定方法 - Qiita](https://qiita.com/schwarz471/items/9b44adfbec006eab60b0)

```
        proxy_set_header HOST $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        location / {
                proxy_pass http://localhost:8000;
        }
```

### Static files
- [Nginxの静的ファイル配信でハマった - shoya\.io](https://shoya.io/ja/posts/nginx-root/)

```
        location /static/ {
                root PARENT_OF_STATIC_DIRECTORY;
        }
```
