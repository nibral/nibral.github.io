---
layout: default
title: 環境構築メモ
---

Index
----

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

Memo
----

findコマンド

* 作成日がn日より前のファイルを列挙
	+ `find ./* -mtime +n`
* 拡張子で絞り込むときは正規表現を使う
	+ `find ./* -regextype posix-egrep -regex "~.*(jpg|mp4)$"`
	+ デフォルトでは正規表現がemacsスタイルなので、いちいちエスケープが必要
	+ posixスタイルにしよう
