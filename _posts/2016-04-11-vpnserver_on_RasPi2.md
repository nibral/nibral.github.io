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

zipを展開して```2016-03-18-raspbian-jessie.img```を手に入れたら、それをmicroSDに書き込む。
今回はLinuxマシンで作業したので、```dd```コマンドを使う。
コマンドは[公式のインストールガイド](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)そのまま。
(```of=```で示される書き込み先は環境により異なるので、先に```df -h```でmicroSDのデバイス名を確認しておく)

    dd bs=4M if=2016-03-18-raspbian-jessie.img of=/dev/sdb

書き込みが終わったら、```sync```を実行してからmicroSDを取り出す。

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
[Zeroconf](https://ja.wikipedia.org/wiki/Zeroconf)の実装である```avahi-daemon```が
標準で動作していて、DHCPで割り当てられたIPに対して```raspberrypi.local```で名前解決ができるようになっている。
今回はこの```avahi-daemon```を活かすため、ルータ側でRasPi(の有線NIC)のMACアドレスに対して
割り当てるIPを固定した。
![](/assets/images/2016-04-11_vpnserver_on_RasPi2/dhcp.jpg)  

補足…Windows10からはmDNSが実装されて```.local```の名前解決ができるようなのだが
([仮想化雑記帳: Windows and Bonjour](http://virtnote.blogspot.jp/2015/06/windows-10-and-bonjour.html))、
手元の環境ではWindows 10 -> RasPiの名前解決ができなかった。RasPi -> Windows 10の名前解決はできるので、
ファイアウォールの設定等を見直す必要があるかもしれない。

RasPiのIPが固定できたら、WAN側からのVPN通信をそのIPに流すように設定する(ポート開放)。
今回利用するLT2P/IPsecでは、UDPの500番と4500番ポートを開ければよい。
![](/assets/images/2016-04-11_vpnserver_on_RasPi2/napt.jpg)  


SoftEther VPNのインストール
----

適当なマシンで[SoftEtherダウンロードセンター](http://softether-download.com/ja.aspx)に行き、
SoftEther VPNの入った```tar.gz```ファイルのURLを確認する。

* コンポーネント:SoftEther VPN Server
* プラットフォーム: Linux
* CPU: ARM EABI (32bit)

URLを確認したら、RasPiへダウンロード。

    wget http://jp.softether-download.com/<中略>/softether-vpnserver-v4.19-9605-beta-2016.03.06-linux-arm_eabi-32bit.tar.gz

あとは公式ドキュメントの[7.3 Linux へのインストールと初期設定](https://ja.softether.org/4-docs/1-manual/7/7.3)に
したがってファイルを展開し、ビルドし、適切な場所に配置し、動作チェックを行ってデーモンとして登録すればよい。
ただし、Raspbianはsystemdを採用しているので、「7.3.8 スタートアップスクリプトへの登録」はスキップして
[Systemd用SoftEther設定ファイル](http://blog.204504byse.info/wiki.cgi?page=Systemd%CD%D1SoftEther%C0%DF%C4%EA%A5%D5%A5%A1%A5%A4%A5%EB)
に従う。

```vpncmd```によるチェックでエラーが出なければ、この時点でVPNサーバが稼働している。
あとは別のWindowsマシンに「SoftEther VPN サーバ管理マネージャ」をインストールし、指示に従って
初期設定を行う。L2TP/IPsecによるアクセスを有効にすることと、DDNSの設定を行うことを忘れないように。
