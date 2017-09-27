---
title: "Amazon Linuxに最新版のnginxをインストールする"
date: 2016-05-21T12:00:00+09:00
---

`amzn-main`リポジトリに登録されてるnginxが古いので、
yumにnginxのリポジトリを追加して最新版をインストールする。

注:インターネットからアクセスするためには、HTTP(80番ポート)の通信を許可するように
セキュリティグループを作成してインスタンスに割り当てる必要がある。詳細は
[Linux インスタンスの Amazon EC2 セキュリティグループ](http://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/using-network-security.html])
を参照。

* `amzn-main`リポジトリの`nginx`を無効化(exclude行を追加)

```
sudo vi /etc/yum.repos.d/amzn-main.repo 
--------
[amzn-main]
name=amzn-main-Base
mirrorlist=http://repo.$awsregion.$awsdomain/$releasever/main/mirror.list
mirrorlist_expire=300
metadata_expire=300
priority=10
failovermethod=priority
fastestmirror_enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-amazon-ga
enabled=1
retries=5
timeout=10
report_instanceid=yes
exclude=nginx
```

* `nginx`リポジトリの定義ファイルを作成

```
sudo vi /etc/yum.repos.d/nginx.repo
--------
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/6/$basearch/
gpgcheck=0
enabled=1
```

* nginxをインストール

```
sudo yum install nginx
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
