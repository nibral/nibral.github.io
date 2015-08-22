---
layout: default
title: めとべや東京#9 メモ
permalink: roommetro-tokyo-9
---

めとべや東京\#9 メモ
====

UWPアプリケーション開発入門
----

* [当日の資料](https://docs.com/nishimura-makot/3022)

* UWP
	+ Windows 10ならどこでもうごく
	+ Desktop,Mobile,IoT,Holo Lens,Xbox,etc.

* WinRTとプラットフォームの構成が似てる
	+ 基本的にUniversal Windows Platformの上で走る

* Bridge for...
	+ (iOS)Obj-CのコードをVSでビルドして実行できる、ただしストーリーボード非対応(?)
	+ (Android)apkを提出して変換
	+ (Web)
	+ (Classic)WPFを変換

* UWPのメリット
	+ ストア配布なので不特定多数に配りやすい
	+ ライバル少ない(人気が…)
	+ タッチ操作は作りやすい

* デメリット
	+ 限定配布は難しい
	+ 複雑なフォームは作りにくい

* Hello,world demo
	+ イベントハンドラ作って動かす感じ

* まとめ
	+ マルチプラットフォーム化が進むよ
	+ Xamarinもらおうね


MADOSMAのこれまでとこれから、とWindows 10
----

* 冒頭

<blockquote class="twitter-tweet" lang="ja"><p lang="ja" dir="ltr">MCJ 平井さんセッション&#10;「Windows 10 を使っている方は…？」 &#10;8 割くらい手が挙がる&#10;「MADOSMA ご利用の方は…？」&#10;8 割くらい手が挙がる&#10;&#10;「異常ですねぇ」 <a href="https://twitter.com/hashtag/%E3%82%81%E3%81%A8%E3%81%B9%E3%82%84?src=hash">#めとべや</a></p>&mdash; ぐらばく (@Grabacr07) <a href="https://twitter.com/Grabacr07/status/634951582031855616">2015, 8月 22</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


* 開発者以外、地方で意外に売れてる

* Win10の動作はほぼOK
	+ 開発レベルではOTAでアップデートも実現されている

* 今年Win10を出すとなると、MSM8909しか選択肢がない
	+ 性能0.7倍!=コスト0.7倍

* MADOSMAの次
	+ 「性能はそのままでサイズを下げてほしい」という日本特有の要望
	+ 日本だけだと売れても10万台程度、ハードベンダーがやりたがらない
	+ 大画面の端末は進行中

* モバイルホットスポット
	+ 事前にPCとリンクしておくと、PC側からホットスポットを起動できる

* Windows 10のリカバリが優秀
	+ `scanstate.exe /apps /ppkg`
	+ `apps.ppkg`

* 開発段階でお蔵入りになった端末
	+ 4.7インチで100g切り
	+ あまりの軽さにモックと勘違いする人続出
	+ 技適等も通っていたが、Band19の掴みが悪く(アンテナサイズの都合?)販売には至らず

Windows 10 Mobile語り
----

* Windows Phone 7
	+ IS12Tのつらい記憶
	+ そもそもOSとして制限がキツすぎた

* Windows Phone 8 暗黒期
	+ docomoが出さない
	+ 相変わらず地図が

* Windows Phone 8 黎明期
	+ 中華フォント問題は開発者側での対応が必要

* Windows 10 Mobile
	+ 地図がBingになってだいぶまともに


Application Insightsでお手軽Windows Phoneアプリ監視
----

* 自分の作ったアプリ監視したい
	+ ガチな監視はいらない
	+ 自分で作るのはおいしくない

* Application Insights in Windows Azure
	+ クラッシュ検出(キャッチされない例外で落ちた時は自動、キャッチした時はAPIから投げる)
	+ `Diagnostics.TelemetryClient`

* どうやって使うか
	+ VSに追加オプションあり
	+ Azureポータルから手動で追加して、発行されたキーをconfigファイルに書いてもよい

* ユーザーとセッション
	+ 1端末1ユーザとして認識される
	+ アプリを起動して終了するまでが1セッション


未来XAMLの1msに泣かないために
----

<iframe src="//www.slideshare.net/slideshow/embed_code/key/jAYWgEuZErUCNK" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/shigure/xaml1ms" title="未来(あす)Xamlの1msに泣かないために" target="_blank">未来(あす)Xamlの1msに泣かないために</a> </strong> from <strong><a href="//www.slideshare.net/shigure" target="_blank">shigure</a></strong> </div>

* Core i7とSnapdragon 410
	+ ベンチマーク的には6倍の性能差
	+ ListViewの処理時間的には16倍

* VS2015のパフォーマンス測定
	+ どこの処理に時間がかかってるのかが測れる(DiskI/O,Network,XAMLlayout)

* XAMLが遅い
	+ パースとレンダリング

* 速くするには
	+ 複雑なレイアウトを避ける、ネストは浅く
	+ 色指定などのパラメータを文字列で渡すとクラスへの変換が走るので遅くなる、Resource化も検討
	+ Resourceにするときは、PageやUserContolのように毎回生成が走るところに置かない
	+ デフォルト値でいいなら、そもそも引数を渡さないのもアリ

* Panelは並べ方で速度が変わる
	+ 複雑なGirdは遅い
	+ Canvasは絶対座標でレンダリングされ、周囲に対する依存がないので高速

* Win8.x向けアプリをWin10で動かすときは地雷を踏むかも
	+ Listview on ScrollViewer

* つねにパフォーマンスを意識する
	+ XAMLが複雑になりすぎてないか
	+ 計測する


LT
----

* Windows 10 IoT
	+ 焦電センサで人がいるか判断するやつ
* KanColleViewerプラグイン
	+ NuGetからパッケージ入れて、IPluginとIToolを実装
* VisualStudio拡張機能
	+ 熊予測
	+ エディタの背景に画像が！
* NotificationsExtentions on Win10
	+ ライブタイルがサクッと
* Office
