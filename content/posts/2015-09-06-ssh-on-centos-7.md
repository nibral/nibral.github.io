---
title: "CentOS7でSSHを使う"
date: 2015-08-22T12:00:00+09:00
---

CentOS7になって、iptablesがfirewalldになったりsystemctl使うようになったりしたので、6.x系から変わったところを中心に。

ユーザ関連
----

インストール中に一般ユーザを作成し、かつそのユーザでsudoコマンドを使いたい場合、インストール完了後に当該ユーザをwheelグループに追加する

	gpasswd -a akari wheel


firewalld
----

結局、firewalldの設定はどこを見ているのか？

1. firewalldが立ち上がると、各NICに対して「ゾーン」が紐付けされる
1. ゾーンには「サービス」や「ポート」、「マスカレードの可否」といったルールが紐付けされている
	+ このルールに従って、パケットを通したり捨てたりする
1. サービスのルールはxmlファイルに保存されていて、必要に応じて編集できる
	+ 初期状態では、`/usr/lib/firewalld/services`に保存されているxmlファイルを参照している
	+ ファイルを編集したいときは直接編集せず、`/etc/firewalld/services`にコピーして書き換える

sshdの標準ルール

	<?xml version="1.0" encoding="utf-8"?>
	<service>
		<short>SSH</short>
		<description>略</description>
		<port protocol="tcp" port="22"/>
	</service>

sshdのポートを変える場合

	cp /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/ssh.xml
	vi /etc/firewalld/services/ssh.xml
		<port protocol="tcp" port="22"/>
		↓
		<port protocol="tcp" port="10022"/>
	systemctl restart firewalld

ただ、手元の環境ではfirewalldの再起動がタイムアウトして失敗する。
設定に問題があるか、サービス間の依存関係に見落としがあるか、単にマシンの性能が足りないか(Celeron N3050でHyper-Vは重い)。
設定が終わったあとにrebootしたらちゃんと動いたのでよし。

SELinux
----

管理ツールのインストール

	yum install policycoreutils-python

sshdが待ち受けるポートとしてTCPの10022番を追加

	semanage port -a -t ssh_port_t -p tcp 10022

ちゃんと登録できたか確認

	semanage port -l | grep ssh


SSH
----

sshdの設定変更

	vi /etc/ssh/sshd_config
		Port 10022
		PermitRootLogin no
		PubkeyAuthentication yes
		PasswordAuthentication yes
		AuthorizedKeysFile .ssh/authorized_keys		

公開鍵認証に使うファイルのパーミッションに過不足があると`Permission denied`になるので注意

* ~/.sshは700
* ~/.ssh/authorized_keysは600

sshd再起動

	systemctl restart sshd.service
	systemctl status sshd.service


その他
----

MSYSgitとWindowsのpingコマンドの相性が悪いのか、出力が中途半端なところで切れたりする  
![](/images/2015-09-06-ssh-on-centos-7/ping.png)  
パケッ
