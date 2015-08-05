---
layout: default
title: 環境構築メモ
---

Index
----

* [Chinachu on CentOS 6.6 + PT3](chinachu.html)
* [PPTP VPN on RTX1000](rtx1000_pptp.html)

Memo
----

findコマンド

* 作成日がn日より前のファイルを列挙
	+ `find ./* -mtime +n`
* 拡張子で絞り込むときは正規表現を使う
	+ `find ./* -regextype posix-egrep -regex "~.*(jpg|mp4)$"`
	+ デフォルトでは正規表現がemacsスタイルなので、いちいちエスケープが必要
	+ posixスタイルにしよう

