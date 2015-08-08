---
layout: default
title: 環境構築メモ
---

Index
----

<<<<<<< HEAD
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
=======
* [Twitterに【メモ】を付けて流したやつ](http://twilog.org/_nibral/search?word=%E3%80%90%E3%83%A1%E3%83%A2%E3%80%91&ao=a)
* [Chinachu on CentOS 6.6 + PT3](chinachu.html)
* [PPTP VPN on RTX1000](rtx1000_pptp.html)
>>>>>>> f89d4fac9973fbd2d9d4f31d5ad7fe1d19713ae0

Memo
----

findコマンド

* 作成日がn日より前のファイルを列挙
	+ `find ./* -mtime +n`
* 拡張子で絞り込むときは正規表現を使う
	+ `find ./* -regextype posix-egrep -regex "~.*(jpg|mp4)$"`
	+ デフォルトでは正規表現がemacsスタイルなので、いちいちエスケープが必要
	+ posixスタイルにしよう
