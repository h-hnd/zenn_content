---
title: "Raspberry Piにハニーポット(dionaea)を入れてみる" # 記事のタイトル
emoji: "" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Raspberry Pi", "dionaea", "honeypot"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
Raspberry PiにDionaeaを入れてみて、どんなアクセスが来るのかを見てみようと思ったので
やってみました。
(取り敢えず投稿しましたが、情報は1年以上前のものです)
(Qiitaにも同じ記事がありますがバックアップも兼ねて持ってきました)

# やった手順

## OS関係

### Raspberry PiにOSを入れる
* raspbian-stretch with Desktopをダウンロードしてきてメモリーカードに入れる
    * Win32 Disk Imager(https://sourceforge.net/projects/win32diskimager/)
    * フォーマットする場合はSDメモリカードフォーマッター(https://www.sdcard.org/jp/downloads/formatter_4/)
        でまずフォーマットする
* イメージを入れたSDカードをセットしてRaspberry Piを起動する
* 最新状態にアップデートする

```bash
$ sudo apt-get update
$ sudo apt-get dist-upgrade
```

### ユーザ関連の設定変更
piユーザが有効だと怖いので、他のユーザを作成した後に無効化しておきます。

* ユーザの作成(ユーザをhoneyとして以降設定)

```bash

$ sudo useradd -m -s /bin/bash honey 
```

* sudo権限の変更

```bash
$ sudo visudo
```

```diff
@@ -20,8 +20,7 @@
 root   ALL=(ALL:ALL) ALL
 
 # Allow members of group sudo to execute any command
 %sudo        ALL=(ALL:ALL) ALL
+honey        ALL=(ALL:ALL) ALL
 
 # See sudoers(5) for more information on "#include" directives:
 ```

```bash
$ sudo vi /etc/sudoers.d/010_pi-nopasswd
```

```diff
@@ -1 +1 @@
-pi ALL=(ALL) NOPASSWD: ALL
+#pi ALL=(ALL) NOPASSWD: ALL
```

* piユーザの無効化

```bash
$ sudo usermod -L pi
```

### sshログインを鍵認証に限定する
一応鍵認証のみのログインにしておきます。

* 鍵を作っておく

```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/honey/.ssh/id_rsa): 
Created directory '/home/honey/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/honey/.ssh/id_rsa.
Your public key has been saved in /home/honey/.ssh/id_rsa.pub.
The key fingerprint is:
===(masked)===============================================
The key's randomart image is:
+---[RSA 2048]----+
| (masked)        |
|                 |
|                 |
|                 |
|                 |
|                 |
|                 |
|                 |
|                 |
+----[SHA256]-----+
```

* sshログインを鍵認証のみにする
    * まずは鍵認証を有効にしてサービス再起動
    * Rootユーザでのログイン無効化も合わせて設定

```bash
$ sudo vi /etc/ssh/sshd_config
```

```diff
@@ -29,15 +29,15 @@
 # Authentication:
 
 #LoginGraceTime 2m
-#PermitRootLogin prohibit-password
+PermitRootLogin no
 #StrictModes yes
 #MaxAuthTries 6
 #MaxSessions 10
 
-#PubkeyAuthentication yes
+PubkeyAuthentication yes
 
 # Expect .ssh/authorized_keys2 to be disregarded by default in future.
-#AuthorizedKeysFile    .ssh/authorized_keys .ssh/authorized_keys2
+AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
 
 #AuthorizedPrincipalsFile none
```

* sshサービスの再起動

```bash
$ sudo service ssh restart
```

* 鍵認証でログイン出来ることを確認する

* パスワードログインの無効化

```bash
$ sudo vi /etc/ssh/sshd_config
```

```diff
@@ -53,8 +53,8 @@
 #IgnoreRhosts yes
 
 # To disable tunneled clear text passwords, change to no here!
-#PasswordAuthentication yes
-#PermitEmptyPasswords no
+PasswordAuthentication no
+PermitEmptyPasswords no
 
 # Change to yes to enable challenge-response passwords (beware issues with
 # some PAM modules and threads)
```

* sshサービスの再起動

```bash
$ sudo service ssh restart
```

## Dionaeaのインストール

Ubuntuだとapt-get installできるのですが、Raspbianでそれが行けるかが分からなかったので
コンパイルして入れることにしました。

### ソースのダウンロード、コンパイル、インストール
* ソースのダウンロード

```bash
$ cd
$ git clone https://github.com/DinoTools/dionaea.git
$ cd  dionaea
```

* 必要なパッケージのインストール

```bash
$ sudo apt-get install \
    build-essential \
    check \
    cmake3 \
    cython3 \
    libcurl4-openssl-dev \
    libemu-dev \
    libev-dev \
    libglib2.0-dev \
    libloudmouth1-dev \
    libnetfilter-queue-dev \
    libnl-dev \
    libpcap-dev \
    libssl-dev \
    libtool \
    libudns-dev \
    python3 \
    python3-dev \
    python3-bson \
    python3-yaml
```
ここで、cmake3が無いと言われたのでcmakeで再チャレンジ。

```bash
$ sudo apt-get install cmake
$ cmake -version
cmake version 3.7.2

CMake suite maintained and supported by Kitware (kitware.com/cmake).
```

バージョンが3以上で入ったので多分大丈夫だろうと言うことで他も再チャレンジ。

```bash
$ sudo apt-get install \
    build-essential \
    check \
    cython3 \
    libcurl4-openssl-dev \
    libemu-dev \
    libev-dev \
    libglib2.0-dev \
    libloudmouth1-dev \
    libnetfilter-queue-dev \
    libnl-dev \
    libpcap-dev \
    libssl-dev \
    libtool \
    libudns-dev \
    python3 \
    python3-dev \
    python3-bson \
    python3-yaml
```
コンパイルしてインストール。

```bash
$ mkdir build
$ cd build
$ cmake -DCMAKE_INSTALL_PREFIX:PATH=/opt/dionaea ..
$ make
$ sudo make install
```

### 設定ファイル
公式サイトにそのまま使えそうな設定ファイルがあったので流用しました。

```config:dionaea.cfg
[dionaea]
download.dir=@LOCALESTATEDIR@/dionaea/binaries/
modules=curl,python,nfq,emu,pcap
processors=filter_streamdumper,filter_emu

listen.mode=getifaddrs
# listen.addresses=127.0.0.1
# listen.interfaces=eth0,tap0

# Country
# ssl.default.c=GB
# Common Name/domain name
# ssl.default.cn=
# Organization
# ssl.default.o=
# Organizational Unit
# ssl.default.ou=

[logging]
default.filename=@LOCALESTATEDIR@/dionaea/dionaea.log
default.levels=all
default.domains=*

errors.filename=@LOCALESTATEDIR@/dionaea/dionaea-errors.log
errors.levels=warning,error
errors.domains=*

[processor.filter_emu]
name=filter
config.allow.0.protocols=smbd,epmapper,nfqmirrord,mssqld
next=emu

[processor.filter_streamdumper]
name=filter
config.allow.0.types=accept
config.allow.1.types=connect
config.allow.1.protocols=ftpctrl
config.deny.0.protocols=ftpdata,ftpdatacon,xmppclient
next=streamdumper

[processor.streamdumper]
name=streamdumper
config.path=@LOCALESTATEDIR@/dionaea/bistreams/%Y-%m-%d/

[processor.emu]
name=emu
config.limits.files=3
#512 * 1024
config.limits.filesize=524288
config.limits.sockets=3
config.limits.sustain=120
config.limits.idle=30
config.limits.listen=30
config.limits.cpu=120
#// 1024 * 1024 * 1024
config.limits.steps=1073741824

[module.nfq]
queue=2

[module.nl]
# set to yes in case you are interested in the mac address  of the remote (only works for lan)
lookup_ethernet_addr=no

[module.python]
imports=dionaea.log,dionaea.services,dionaea.ihandlers
sys_paths=default
service_configs=@SYSCONFDIR@/dionaea/services-enabled/*.yaml
ihandler_configs=@SYSCONFDIR@/dionaea/ihandlers-enabled/*.yaml

[module.pcap]
any.interface=any
```
### ユーザ作成と所有者変更
dionaeaユーザを作成して、これまでrootの持ち物だったDionaea一式の
所有者を変更します。

```bash
$ sudo useradd -m dionaea
$ cd /opt
$ sudo chown -R dioanaea
```

ここまでやれば取り敢えずDionaeaは動くはずです。

## DionaeaFRのインストール
視覚化ツールがあるので入れてみます。

### 各パッケージのインストール

```bash
$ sudo pip install Django
$ sudo pip install pygeoip
$ sudo pip install django-pagination
$ sudo pip install django-tables2
$ sudo pip install django-compressor
$ sudo pip install django-htmlmin
$ sudo pip install django-filter
$ sudo pip install uwsgi
$ cd
$ git clone https://github.com/benjiec/django-tables2-simplefilter
$ cd django-tables2-simplefilter
$ sudo python setup.py install
$ cd
$ git clone git://git.bro-ids.org/pysubnettree.git
$ cd pysubnettree
$ sudo python setup.py install
$ cd
$ wget https://nodejs.org/dist/v8.11.2/node-v8.11.2.tar.gz
$ tar zxvf node-v8.11.2.tar.gz
$ cd node-v8.11.2/
$ ./configure
$ make
$ sudo make install
$ sudo npm install -g less
$ sudo apt-get install python-netaddr
```

### DionaeaFRのインストール

* ソースの取得

```bash
$ cd
$ git clone https://github.com/rubenespadas/DionaeaFR.git
```

* 地理情報を取得してDionaeaFRディレクトリに格納

```bash
$ wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
$ wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz
$ gunzip GeoLiteCity.dat.gz
$ gunzip GeoIP.dat.gz
$ mv GeoIP.dat DionaeaFR/DionaeaFR/static
$ mv GeoLiteCity.dat DionaeaFR/DionaeaFR/static
```

* DionaeaFRディレクトリを移動して所有者を変更

```bash
$ sudo mv DionaeaFR /opt/
$ cd /opt
$ sudo chown -R dionaea:dionaea DionaeaFR
```

これでDionaeaFRも動くはず…です。

## 外部からDionaeaFRへのアクセス無効化

外からのアクセスをかなり緩くするので、node.jsで受け付けているDionaeaFRは
localhostでしか見られない様に変更します。

* UFWのインストール

```bash
$ sudo apt-get install ufw
```

* ルール設定

あくまでハニーポット用に意図的に緩くするため、あえてデフォルトをallowにします。

```bash

```


## いざ実行…!

* Dionaeaの実行

```bash
$ sudo /opt/dionaea/bin/dionaea -D -u dionaea -c /opt/dionaea/etc/dionaea/dionaea.cfg -w /opt/dionaea -p /opt/dionaea/var/run/dionaea.pid
```

* DionaeaFRの実行

```bash
$ cd ~/DionaeaFR
$ sudo python manage.py collectstatic
$ sudo python manage.py runserver 0.0.0.0:53521
```

# 参考にしたサイト
* 公式サイト(https://dionaea.readthedocs.io/en/latest/installation.html#rd-party-packages)
* VPSにハニーポット(Dionaea)を入れてみた(http://takahoyo.hatenablog.com/entry/2014/05/26/023409)
* VPSにDionaeaログ解析ツール、DionaeaFRを入れてみた(http://takahoyo.hatenablog.com/entry/2014/06/07/233059)
