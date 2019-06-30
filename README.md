# gee4-bath-server

ギークハウス新宿四ツ谷の風呂センサーの親機サーバーとして使用している raspberry pi をセットアップする Ansible Playbook

## Requirements

- Ansible 2.8 or later

## Usage

``` console
$ ansible-playbook -i hosts_prod gee4-bath-server.yaml -u <ユーザー名> -K
```

## Development

QEMUを使ってx86_64上でraspbianのテスト環境を構築する方法

### Requirements

- qemu: https://www.qemu.org/download/
- qemu-rpi-kernel: https://github.com/dhruvvyas90/qemu-rpi-kernel
- Raspbian OS: https://www.raspberrypi.org/downloads/raspbian/

### QEMU起動

``` console
$ qemu-system-arm \
  -M versatilepb \
  -cpu arm1176 \
  -m 256 \
  -hda <Raspbin OS の img ファイル> \
  -net nic \
  -net user,hostfwd=tcp::5022-:22 \
  -dtb <qemu-rpi-kernel でダウンロードした versatile-pb.dtb> \
  -kernel <qemu-rpi-kernel でダウンロードした kernel-qemu-* の中で適したやつ> \
  -append 'root=/dev/sda2 panic=1' \
  -no-reboot
```

下の組み合わせで動作確認できた

- hda: 2019-04-08-raspbian-stretch-lite.img
- kernel: qemu-rpi-kernel/kernel-qemu-4.14.79-stretch

### SSH接続するための準備

QEMUの画面からraspbianにログインしてSSHを起動させる。
初期設定では、

- user: pi
- password: raspberry

``` console
$ sudo systemctl start ssh
```

終わったらログアウト

### ホストOSからraspbianにSSH接続

``` console
$ ssh pi@localhost -p 5022
```

接続できたら、Ansibleで接続するようのユーザーアカウントを作成しておく。
（piユーザーはPlaybookで消す）

``` console
# useradd -m -G sudo -s /bin/bash ユーザー名
# passwd ユーザー名
```

できたらゲストOSからログアウトする

### 公開鍵を設置する

あらかじめ、ゲストOSへのSSH接続に使用したいキーペアを用意しておく。

``` console
$ ssh-copy-id -i 公開鍵へのPATH -p 5022 pi@localhost
```
