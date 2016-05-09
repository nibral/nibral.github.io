---
layout: default
title: Raspberry Pi 2をVPNサーバに仕立てる
permalink: vpnserver_on_RasPi2
---

Raspberry Pi 2をVPNサーバに仕立てる
====

はじめに
----

購入直後にひとしきり遊んで放置されていたRasberry Pi 2 Model Bを発見したので、

* 公衆無線LAN利用時のセキュリティ向上
* 自宅NASへのアクセス

を目的としてVPNサーバを立てた。VPNソフトウェアは[SoftEther VPN](https://ja.softether.org/)。
また、

* 接続するLANでDHCPが提供されている
* 作業はすべてSSH経由で行い、USBキーボードとマウスを使わない(適当なHDMI対応モニタがなかったので)
* SoftEther VPNの設定は、別のWindowsマシンから「SoftEther VPN サーバ管理マネージャ」を使って行う

ものとする。

やるべきこと
----

* OSのインストール
* Raspbianのセットアップ
* ネットワーク設定
* SoftEther VPNのインストール

OSのインストールと初期セットアップ
----

まず、[Raspberry Pi公式のRaspbianダウンロードページ](https://www.raspberrypi.org/downloads/raspbian/)から
OSイメージをダウンロードする。Raspbianには通常版とLite版があるが、Lite版でインストールされないパッケージに
依存するソフトウェアがあると面倒なので今回は通常版を選択。

zipを展開して`2016-03-18-raspbian-jessie.img`を手に入れたら、それをmicroSDに書き込む。
今回はLinuxマシンで作業したので、`dd`コマンドを使う。
コマンドは[公式のインストールガイド](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)そのまま。
(`of=`で示される書き込み先は環境により異なるので、先に`df -h`でmicroSDのデバイス名を確認しておく)

    dd bs=4M if=2016-03-18-raspbian-jessie.img of=/dev/sdb

書き込みが終わったら、`sync`を実行してからmicroSDを取り出す。

Raspbianのセットアップ
----

OSイメージを書き込んだmicroSDをRasPiに差し込み、LANケーブルと電源を接続する。
DHCP環境であれば、数分もするとIPが割り当てられてSSHでアクセスできるようになる。
本来はRasPiに割り当てられたIPを探すために必要があるが、今回はルータでIPの割り当てが確認できたので不要だった。
![](/assets/images/2016-04-11_vpnserver_on_RasPi2/netmap.jpg)  

IPがわかったらSSHで接続し(ユーザ名pi、パスワードraspberry)、Raspberry Piの設定を行う。

    raspi-config

少なくとも、「1. Expand Filesystem」の実行と「4. i18n Options」のロケールとタイムゾーン設定は必須。
終わったら一旦再起動し、続いて各種アップグレードを済ませる。

    apt-get update
    apt-get upgrade
    rpi-update

ファームウェアの更新が行われるためここでも再起動。


ネットワーク設定
----

サーバとして動作させる関係上、RasPiのIPは固定値が望ましい。通常はサーバ側に固定IPを設定するが、RasPiでは
[Zeroconf](https://ja.wikipedia.org/wiki/Zeroconf)の実装である`avahi-daemon`が
標準で動作していて、DHCPで割り当てられたIPに対して`raspberrypi.local`で名前解決ができるようになっている。
今回はこの`avahi-daemon`を活かすため、ルータ側でRasPi(の有線NIC)のMACアドレスに対して
割り当てるIPを固定した。
![](/assets/images/2016-04-11_vpnserver_on_RasPi2/dhcp.jpg)  

補足…Windows10からはmDNSが実装されて`.local`の名前解決ができるようなのだが
([仮想化雑記帳: Windows and Bonjour](http://virtnote.blogspot.jp/2015/06/windows-10-and-bonjour.html))、
手元の環境ではWindows 10 -> RasPiの名前解決ができなかった。RasPi -> Windows 10の名前解決はできるので、
ファイアウォールの設定等を見直す必要があるかもしれない。

RasPiのIPが固定できたら、WAN側からのVPN通信をそのIPに流すように設定する(ポート開放)。
今回利用するLT2P/IPsecでは、UDPの500番と4500番ポートを開ければよい。
![](/assets/images/2016-04-11_vpnserver_on_RasPi2/napt.jpg)  


SoftEther VPNのインストール
----

適当なマシンで[SoftEtherダウンロードセンター](http://softether-download.com/ja.aspx)に行き、
SoftEther VPNの入った`tar.gz`ファイルのURLを確認する。

* コンポーネント:SoftEther VPN Server
* プラットフォーム: Linux
* CPU: ARM EABI (32bit)

URLを確認したら、RasPiへダウンロード。
```
wget http://jp.softether-download.com/<中略>/softether-vpnserver-v4.19-9605-beta-2016.03.06-linux-arm_eabi-32bit.tar.gz
```
あとは公式ドキュメントの[7.3 Linux へのインストールと初期設定](https://ja.softether.org/4-docs/1-manual/7/7.3)に
したがってファイルを展開し、ビルドし、適切な場所に配置し、動作チェックを行ってデーモンとして登録すればよい。
ただし、Raspbianはsystemdを採用しているので、「7.3.8 スタートアップスクリプトへの登録」はスキップして
[Systemd用SoftEther設定ファイル](http://blog.204504byse.info/wiki.cgi?page=Systemd%CD%D1SoftEther%C0%DF%C4%EA%A5%D5%A5%A1%A5%A4%A5%EB)
に従う。

`vpncmd`によるチェックでエラーが出なければ、この時点でVPNサーバが稼働している。
あとは別のWindowsマシンに「SoftEther VPN サーバ管理マネージャ」をインストールし、指示に従って
初期設定を行う。L2TP/IPsecによるアクセスを有効にすることと、DDNSの設定を行うことを忘れないように。

追記
----

RasPi内臓の有線LANインターフェース(eth0)にSoftether VPNの仮想HUBをブリッジすると、
Liunx側の制約でVPNサーバが提供する他のサービス(httpサーバ等)にVPNクライアントからアクセス出来ない。

>Linuxオペレーティングシステム内部での制限事項により、VPN側(仮想HUB側)からローカルブリッジしている
>LANカードに割り当てられるIPアドレスに対して通信を行うことはできません。この制限はSoftEtherVPNが原因ではなく、
>Linuxの内部構造に原因があります。もしVPN側(仮想HUB側)からLinuxでローカルブリッジに使用している
>コンピュータ本体と、何らかの通信を行いたい場合(たとえばVPN Server/VPN BridgeサービスとHTTPサーバー
>サービスを両方動作させており、VPN側からもサーバーサービスにアクセスさせたい場合)は、ローカルブリッジ用の
>LAN カードを用意して接続し、そのLANカードと既存のLANカードの両方を物理的に同じセグメントに接続してください
>\([3.6.11 Linux, FreeBSD, Solaris, Mac OS X オペレーティングシステムで「ローカルブリッジ機能」を使用する場合の注意事項](https://ja.softether.org/4-docs/1-manual/3/3.6)\)

この問題を解決するためには、以下の2つの方法がある。

1. USB-Ethernetアダプタ(eth1)を増設して仮想HUBのブリッジ専用にする
    + VPN以外のサービスにはeth1→eth0という形で一回外に出て戻ってくる経路になる
    + シンプルだが、IPを2つ専有する上にアダプタとケーブルが邪魔
2. VPN用の仮想インターフェース(tap)を作成し、ブリッジインターフェース(br0)を介して外部及びeth0と接続する
    + eth0のIPをbr0に移し、eth0はbr0へOS内部で接続する
    + tapデバイスはサーバ管理マネージャの「ローカルブリッジ設定」で名前を指定して作成できる

`systemd-networkd`環境下で方法2を用いる場合の手順は以下のとおり。
*リモートで作業する場合、設定をミスるとIPの割り当てられているインターフェースが一つもない状態になって詰むので、*
*一時的にUSB-Ethernetアダプタを接続などして回線を確保しておくほうが安全。*

* ブリッジインターフェースの作成

```
/etc/systemd/network/br0.netdev
```
```
[NetDev]
Name=br0
Kind=bridge
```

* ブリッジのプロファイルを作成

```
/etc/systemd/network/br0.interface
```
```
[Match]
Name=br0

[Network]
DHCP=ipv4
```

* eth0をブリッジに接続

```
/etc/systemd/network/eth0.interface
```
```
[Match]
Name=eth0

[Network]
Bridge=br0
```

* `systemd-networkd`の再起動

```
sudo systemctl restart systemd-networkd.service
```

* `vpnserver`のスタートアップスクリプトを修正して再起動

```
[Unit]
Description=SoftEther VPN Server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/vpnserver/vpnserver start
ExecStop=/usr/local/vpnserver/vpnserver stop

# 追記(tap_vpnの部分は管理マネージャで指定した名前に合わせる)
ExecStartPost=/bin/sleep 5 ; /sbin/brctl addif br0 tap_vpn

[Install]
WantedBy=multi-user.target
```
```
sudo systemctl daemon-reload
sudo systemctl stop vpnserver
sudo systemctl start vpnserver
```
