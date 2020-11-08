---
canonical_url: ./
title: 'スニペット: Docker + Docker Compose on Vagrant Ubuntu Server'
# og_image:
# twitter_card: summary_large_image
og_description: 'VagrantでUbuntu Server + Docker + Docker Compose環境をセットアップする'
date: '2020-10-22 16:30:00'
draft: false
category: スニペット
tags:
  - Vagrant
  - VirtualBox
  - Ubuntu
  - Docker
  - Docker Compose
---
# スニペット: Docker + Docker Compose on Vagrant Ubuntu Server

- [Downloads – Oracle VM VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Downloads | Vagrant by HashiCorp](https://www.vagrantup.com/downloads.html)
- [Vagrant box ubuntu/bionic64 - Vagrant Cloud](https://app.vagrantup.com/ubuntu/boxes/bionic64)
- [Install Docker Engine on Ubuntu | Docker Documentation](https://docs.docker.com/engine/install/ubuntu/)
- [Install Docker Compose | Docker Documentation](https://docs.docker.com/compose/install/)

kvm (qemu)を使うこともできるらしいが、対応したboxを用意する必要があるのでVirtualBoxを使う。

- [KVM用仮想マシンをVagrantで手軽に作る | さくらのナレッジ](https://knowledge.sakura.ad.jp/2535/)

## Commands

```shell
# 起動
vagrant up

# 削除
vagrant destroy

# ssh
vagrant ssh

# 停止（シャットダウン）
vagrant halt
```

## Vagrantfile
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.hostname = "ubuntu-docker"

  # config.vm.network "forwarded_port", guest: 80, host: 8080
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "ubuntu-docker"
    vb.memory = "1024"
    # vb.gui = true
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update

    apt-get install -y \
      apt-transport-https \
      ca-certificates \
      curl \
      gnupg-agent \
      software-properties-common

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

    add-apt-repository -y \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"

    apt-get update

    apt-get install -y \
      docker-ce \
      docker-ce-cli \
      containerd.io

    curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose

    adduser vagrant docker
  SHELL
end
```
