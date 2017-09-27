---
title: "Surface 3で402KCのBluetoothテザリングが使いたかった"
date: 2015-12-24T12:00:00+09:00
---

**追記:初出時は「Bluetoothテザリングを使う」というタイトルでしたが、いざ外で使ってみたところ非常に遅い上に不安定というどうしようもない挙動だったのでこの方法はお勧めしません。**

注:Windows 8.1でのお話です

問題:402KCとペアリングできない
----

Surface 3で402KCのBluetoothテザリングを使うためには、PAN(Personal Area Network)プロファイルでペアリングしなければならない。
当初やろうとしていた手順は、

1. 402KCでBluetoothテザリングの機器登録画面を開き、新規登録が行える状態にする  
![](/images/2015-12-24-tethering-surface3-with-402kc/402kc-ready.jpg)  
2. Surface 3でチャームを出し、「設定」→「PC設定の変更」→「PCとデバイス」→「Bluetooth」を開いて402KCとペアリングを始める  
![](/images/2015-12-24-tethering-surface3-with-402kc/metro-ready.png)  
3. 双方の端末に表示されたパスコードが一致していることを確認して続行する  
![](/images/2015-12-24-tethering-surface3-with-402kc/metro-pairing.png)  
![](/images/2015-12-24-tethering-surface3-with-402kc/402kc-pairing.jpg)

ところが、これだと402KC側で「登録中」画面のままタイムアウトで失敗してしまう。  
![](/images/2015-12-24-tethering-surface3-with-402kc/402kc-failed.jpg)  
このとき、Surface 3側ではペアリング完了の表示になっている。  
![](/images/2015-12-24-tethering-surface3-with-402kc/metro-success.png)  
正確な原因はわからないが、Surface 3上の表示がヘッドセットのアイコン(HFP？)になっているところを見ると、
Surface 3がペアリング後にPANプロファイルの要求(のようなもの)を出しておらず、402KCがこの要求を待ち続けてしまってタイムアウトになると推測。

解決策:デバイスとプリンターからペアリングする
----

「PANの要求が出ないのが問題なら出せばいい」ということでWindowsでのBTテザリングの手順とかを調べたところ、
コントロールパネルの「デバイスとプリンター」から操作することでうまくペアリングできた。以下手順。

1. 402KCでBluetoothテザリングの機器登録画面を開き、新規登録が行える状態にする  
![](/images/2015-12-24-tethering-surface3-with-402kc/402kc-ready.jpg)  
2. Surface 3で「デバイスとプリンター」を開く  
![](/images/2015-12-24-tethering-surface3-with-402kc/win-devlist.png)  
3. 画面左上の「デバイスの追加」を押し、402KCを選んで「次へ」  
![](/images/2015-12-24-tethering-surface3-with-402kc/win-devselect.png)  
4. パスコードの一致を確認して続行する  
![](/images/2015-12-24-tethering-surface3-with-402kc/win-pairing.png)  
![](/images/2015-12-24-tethering-surface3-with-402kc/402kc-pairing.jpg)  
5. Surface 3でペアリング処理が行われ、402KCも「登録中」の表示になる  
![](/images/2015-12-24-tethering-surface3-with-402kc/win-success.png)  
![](/images/2015-12-24-tethering-surface3-with-402kc/402kc-processing.jpg)  
6. **402KCの処理がタイムアウトになる前に**「デバイスとプリンター」上で402KCを選択し、画面上の「接続方法」から「アクセスポイント」を押す。  
![](/images/2015-12-24-tethering-surface3-with-402kc/win-pan-enable.png)  
7. 「接続に成功しました」というメッセージが出て、402KCに詳細情報が表示される(PAN対応機器として認識されていることがわかる)。
![](/images/2015-12-24-tethering-surface3-with-402kc/win-pan-success.png)  
![](/images/2015-12-24-tethering-surface3-with-402kc/402kc-success.jpg)  

402KCへの登録さえ成功すれば、次からは「402KCでBTテザリングをオンにする」→「Surface 3からPANで接続」するだけで接続できる。
ping100ms越えは当たり前、速度も良くて200kbps前後の微妙な回線ではあるけれど、ないよりマシということで。
