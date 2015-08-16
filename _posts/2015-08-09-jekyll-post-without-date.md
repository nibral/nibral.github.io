---
layout: default
title: Jekyllで日付に基づいたディレクトリを作らない
permalink: jekyll-post-without-date
---


Jekyllで日付に基づいたディレクトリを作らない
====

`_posts`ディレクトリに記事を入れるとき、ファイル名に`YYYY-MM-DD-なんとか-かんとか.md`の形式で日付を入れないといけない。
ページのコンパイル時、この日付をもとに


    ＿site
     |-YYYY
       |-MM
         |-DD
           |-なんとか-かんとか.html

という構造ができて、`example.com/YYYY/MM/DD/なんとか-かんとか`で見えるようにしてくれる。

また、日付の情報を持たせたくない場合、Front-Formatterに`permalink`属性を追加するとその名前でリンクを作ってくれる。
例えば、

    ----
    layout: default
    title: すごいメモ
    permalink: supermemo
    ----

とすると、`example.com/supermemo`で見えるようになる。
ファイル名の日付は適当に入れておけばOK。雑多なメモ置き場として使う時にべんり。
