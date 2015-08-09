---
layout: default
title: Chinachu on CentOS 6.6 + PT3
permalink: chinachu-on-centos
---

Chinachu on CentOS 6.6 + PT3
====

事前準備
----

開発ツールを入れる

	sudo yum install yum-plugin-priorities
	sudo yum install kernel-devel
	sudo yum install gcc gcc-c++ make patch

gitはソースから新しいのを入れる

	sudo yum install zlib-devel tcl-devel tk-devel gettext-devel curl-devel expat-devel openssl-devel
	wget https://www.kernel.org/pub/software/scm/git/git-2.3.4.tar.gz
	tar zxvf git-2.3.4.tar.gz
	cd git-2.3.4
	./configure --with-curl=/usr/bin --prefix=/usr/local
	make prefix=/usr/local all
	sudo make prefix=/usr/local install

カードリーダー関連
----

	yum install pcsc-lite pcsc-lite-devel pcsc-lite-libs
	yum install ccid

perl-gtkはRPMForgeから入れる

	wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
	sudo rpm --import http://apt.sw.be/RPM-GPG-KEY.dag.txt
	sudo rpm -K rpmforge-release-0.5.3-1.el6.rf.\*.rpm (Verify)
	sudo rpm -K rpmforge-release-0.5.3-1.el6.rf.\*.rpm (Install)
	yum install --enablerepo=rpmforge -y perl-Gtk2

pcsc-perlはソースから

	wget http://ludovic.rousseau.free.fr/softwares/pcsc-perl/pcsc-perl-1.4.13.tar.bz2
	tar xf pcsc-perl-1.4.13.tar.bz2
	cd pcsc-perl-1.4.13
	perl Makefile.PL PREFIX=/usr SITELIBEXP=/usr/share/perl5 SITEARCHEXP=/usr/lib64/perl5
	make
	sudo make install

pcsc-toolsもソースから

	wget http://ludovic.rousseau.free.fr/softwares/pcsc-tools/pcsc-tools-1.4.22.tar.gz
	tar xf pcsc-tools-1.4.22.tar.gz
	cd pcsc-tools-1.4.22
	make
	sudo make install DESTDIR=/usr

サービス設定

	sudo service messagebus start
	sudo service haldaemon start
	sudo service pcscd start
	sudo chkconfig messagebus on
	sudo chkconfig haldaemon on
	sudo chkconfig pcscd on

openctが競合するので止める

	sudo service openct stop
	sudo chkconfig openct off

`pcsc_scan`を実行して動作チェック

* `Japanese Chijou Digital B-CAS Card (pay TV)`が出ればOK
* `Ctrl+C`で終了

arib25
----

	git clone git://github.com/stz2012/libarib25.git
	make
	sudo make install

recpt1
----

	git clone https://github.com/stz2012/recpt1.git
	cd ./recpt1/recpt1
	./autogen.sh
	./configure --enable-b25
	make
	sudo make install

動作チェック

	recpt1 --b25 27 10 test.ts

PT3ドライバ
----

標準ドライバを無効化

	vi /etc/modprobe.d/blacklist.conf
		"blacklist earth_pt1"を末尾に追加

インストール

	git clone git://github.com/m-tsudo/pt3.git
	cd pt3
	make
	sudo make install

再起動して動作チェック

	sudo reboot now
	ls /dev/ | grep pt3
		pt3video0-3が出ればOK
	recpt1 --b25 --strip 11 10 /var/test.ts --device /dev/pt3video2

ライブラリが読めないエラーが出た

	ldd /usr/local/bin/recpt1	読めないライブラリを確認
	sudo vi /etc/ld.so.conf		"/usr/local/lib"を末尾に追加
	sudo ldconfig
	ldd /usr/local/bin/recpt1

cannot start b25 decorderが出た

* pcscdを再起動してみる

chinachu
----

yasmはepelから入れる

	wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
	sudo rpm -Uvh epel-release-6-8.noarch.rpm
	sudo vi /etc/yum.repos.d/epel.repo
		[epel]をenabled=0に
	sudo yum install --enablerepo=epel yasm yasm-devel

chinachuユーザをつくる

	yum install autoconf automake libtool
	sudo useradd chinachu
	sudo su chinachu
	cd

gitにパスを通す

	vi .bash_profile に/usr/local/binを追記
	source .bash_profile

chinachuのダウンロード

	git clone git://github.com/kanreisa/Chinachu.git ~/chinachu
	cd chinachu

pkgconfigにもパスを通す

	export PKG_CONFIG_PATH=/home/chinachu/chinachu/usr/lib/pkgconfig
	echo $PKG_CONFIG_PATH

インストーラ実行

	./chinachu installer
		1) Auto (full)
	./chinachu service operator initscript > /tmp/chinachu-operator
	./chinachu service wui initscript > /tmp/chinachu-wui
	exit(もとのユーザに戻る)

サービス登録

	mv /tmp/chinachu-operator /tmp/chinachu-wui /etc/init.d/
	chown root:root /etc/init.d/chinachu-operator /etc/init.d/chinachu-wui
	chmod +x /etc/init.d/chinachu-operator /etc/init.d/chinachu-wui
	chkconfig -–add chinachu-operator
	chkconfig –-add chinachu-wui
	chkconfig chinachu-operator on
	chkconfig chinachu-wui on

参考
----

* [PT3の設定(CentOS6)](http://d.hatena.ne.jp/kt_hiro/20130113/1358051168)
* [recpt1のインストール](http://memo.saitodev.com/home/linux_pt2/recpt1/)
