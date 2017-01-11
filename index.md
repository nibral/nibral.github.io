---
layout: index
title: nibral.github.io
---

nibral.github.io
====

* [GitHub](https://github.com/nibral)
* [Gist](https://gist.github.com/nibral)
* [Qiita](http://qiita.com/nibral)

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

* 作成日がn日より前のファイルを列挙
    * `find ./* -mtime +n`
* 拡張子で絞り込むときは正規表現を使う
	* `find ./* -regextype posix-egrep -regex "~.*(jpg|mp4)$"`
	* デフォルトでは正規表現がemacsスタイルなので、いちいちエスケープが必要
	* posixスタイルにしよう
