---
title: 'Ubuntu 22.04, コマンドラインでパスワードをハッシュ化する（htpasswd bcrypt, doveadm SHA-512, openssl SHA-512）'
date: '2023-11-01T23:45:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Ubuntu
tags:
  - Ubuntu
  - Authentication
---
# Ubuntu 22.04, コマンドラインでパスワードをハッシュ化する（htpasswd bcrypt, doveadm SHA-512, openssl SHA-512）

## htpasswd + bcrypt

```shell
$ sudo apt install apache2-utils

$ sudo apt list --installed apache2-utils
apache2-utils/jammy-updates,now 2.4.52-1ubuntu4.6 amd64 [installed]
```

```shell
$ htpasswd -nB myuser
New password: password
Re-type new password: password
myuser:$2y$05$aw7xqaxQ207POKX8bavQleo52mb1jxRT7WenwvXqW21FA4wnygHjq
```

- [password - Compute bcrypt hash from command line - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/307994/compute-bcrypt-hash-from-command-line)

## doveadm + SHA-512

```shell
$ sudo apt install dovecot-core

$ sudo apt list --installed dovecot-core
dovecot-core/jammy-updates,now 1:2.3.16+dfsg1-3ubuntu2.2 amd64 [installed]
```

```shell
$ doveadm pw -s SHA512-CRYPT
Enter new password: password
Retype new password: password
{SHA512-CRYPT}$6$jtT1Mdke./dCtWSp$5ptqP0pgduBjHiRHCfh0nrWstAI46Jmytf88VlrJgpMsBPSNVhFG1cdgxkHVAXLporwb0d9pcYskAfFPqdtEy1
```

- [security - How to create SHA512 password hashes on command line - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/52108/how-to-create-sha512-password-hashes-on-command-line)

## openssl + SHA-512

```shell
$ sudo apt list --installed openssl
openssl/jammy-updates,jammy-security,now 3.0.2-0ubuntu1.12 amd64 [installed]

$ openssl version
OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
```

```shell
$ openssl passwd -6
Password: password
Verifying - Password: password
$6$TmbOcErpIJRy.gWH$phJqAZzmdBj206D4kQEJNEq638KJyUwcnLbK61yBQeVsImjIg0SYXeClDjyMFCVCAvjqeNszhwtz1aUtTPKo70
```

- [security - How to create SHA512 password hashes on command line - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/52108/how-to-create-sha512-password-hashes-on-command-line)
- [LinuxでSHA-512のパスワードハッシュ作成方法まとめ #Python - Qiita](https://qiita.com/yumenomatayume/items/2c77ec52e7b2257f6800)
