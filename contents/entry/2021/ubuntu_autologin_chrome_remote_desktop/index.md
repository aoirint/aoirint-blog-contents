
# Ubuntu 20.04, 自動ログイン＋Chrome Remote Desktopの起動時に繰り返し認証を求められる

Chrome Remote Desktopを導入したUbuntu 20.04に自動ログインを設定すると、
初回の認証はkeyringのロックを解除する認証だが、ほかにも5-6回程度繰り返し認証を求められる。

以下のようなメッセージが表示される。

```shell
Authentication is required to create a color managed device
```

以下のファイルのうち、いずれかを作成することで、追加の認証をスキップできる。

## /etc/polkit-1/localauthority.conf.d/02-allow-colord.conf

- <https://god-support.blogspot.com/2019/11/ubuntu1804-xrdp-authentication-is.html>

```javascript
polkit.addRule(function(action, subject) {
    if ((
            action.id == "org.freedesktop.color-manager.create-device" ||
            action.id == "org.freedesktop.color-manager.create-profile" ||
            action.id == "org.freedesktop.color-manager.delete-device" ||
            action.id == "org.freedesktop.color-manager.delete-profile" ||
            action.id == "org.freedesktop.color-manager.modify-device" ||
            action.id == "org.freedesktop.color-manager.modify-profile"
        )
        && subject.isInGroup("{users}")
    ) {
        return polkit.Result.YES;
    }
});
```

## /etc/polkit-1/localauthority/50-local.d/45-allow-colord.pkla

- <https://www.cagylogic.com/archives/2021/03/23145121/11743.php>

```conf
[Allow Colord all Users]
Identity=unix-user:*
Action=org.freedesktop.color-manager.create-device;org.freedesktop.color-manager.create-profile;org.freedesktop.color-manager.delete-device;org.freedesktop.color-manager.delete-profile;org.freedesktop.color-manager.modify-device;org.freedesktop.color-manager.modify-profile
ResultAny=no
ResultInactive=no
ResultActive=yes
```
