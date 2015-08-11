---
layout: default
title: メモ置き場
---

nibral.github.io
====

- [Twitter](https://twitter.com/_nibral)
    - [技術メモ](http://twilog.org/_nibral/search?word=%E3%80%90%E3%83%A1%E3%83%A2%E3%80%91&ao=a)
- [GitHub](https://github.com/nibral)

Posts
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

- 作成日がn日より前のファイルを列挙
    - `find ./* -mtime +n`
- 拡張子で絞り込むときは正規表現を使う
	- `find ./* -regextype posix-egrep -regex "~.*(jpg|mp4)$"`
	- デフォルトでは正規表現がemacsスタイルなので、いちいちエスケープが必要
	- posixスタイルにしよう
