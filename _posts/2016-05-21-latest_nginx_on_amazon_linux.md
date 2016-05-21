---
layout: default
title: Amazon Linuxに最新版のnginxをインストールする
permalink: latest_nginx_on_amazon_linux
---

Amazon Linuxに最新版のnginxをインストールする
====

`amzn-main`リポジトリに登録されてるnginxが古いので、
yumにnginxのリポジトリを追加して最新版をインストールする。

注:インターネットからアクセスするためには、HTTP(80番ポート)の通信を許可するように
セキュリティグループを作成してインスタンスに割り当てる必要がある。詳細は
[Linux インスタンスの Amazon EC2 セキュリティグループ](http://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/using-network-security.html])
を参照。

* リポジトリの定義ファイルを作成

```
sudo vi /etc/yum.repos.d/nginx.repo
--------
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/6/$basearch/
gpgcheck=0
enabled=1
```

* nginxをインストール(`amzn-main`リポジトリ無効、`nginx`有効)

```
sudo yum install --disablerepo=amzn-main --enablerepo=nginx nginx
```

* サービス起動、自動起動オン

```
sudo service nginx start
sudo chkconfig nginx on
```

* 動作確認

`http://<インスタンスのIP/>`にアクセスして、下のようなページが出ればOK  
![](/assets/images/2016-05-21-latest_nginx_on_amazon_linux/welcome.png)  
設定ファイルは`/etc/nginx/conf.d/`に置く。
