---
title: 'HashiCorpのapt GPGキーを更新する'
date: '2023-01-25T04:00:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Ubuntu
tags:
  - Ubuntu
  - apt
  - HashiCorp
---
# HashiCorpのapt GPGキーを更新する

```shell
$ sudo apt update

Err:2 https://apt.releases.hashicorp.com focal InRelease
  The following signatures couldn't be verified because the public key is not available: NO_PUBKEY AA16FCBCA621E701

W: An error occurred during the signature verification. The repository is not updated and the previous index files will be used. GPG error: https://apt.releases.hashicorp.com focal InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY AA16FCBCA621E701
W: Failed to fetch https://apt.releases.hashicorp.com/dists/focal/InRelease  The following signatures couldn't be verified because the public key is not available: NO_PUBKEY AA16FCBCA621E701
W: Some index files failed to download. They have been ignored, or old ones used instead.
```

- [https://developer.hashicorp.com/terraform/cli/install/apt](https://developer.hashicorp.com/terraform/cli/install/apt)

```shell
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint
```
