---
title: Windowsからvagrant-digitaloceanを使う
date: 2015-08-16T12:00:00+09:00
---

Digital Oceanのクーポンが余ってたので、Vagrantから使ってみる。
ローカルに仮想環境を立てないのでVirtualBoxはインストール不要、
持ち運ぶ端末が仮想化をサポートしない(Surface 3とか)場合も安心。

ConEmuの設定
----

今回は、UNIXのツール群が使える環境としてGit BashをConEmuから呼ぶことにする。

まず、メニューの`Settings...`から`Startup`→`Tasks`を開く。
左下の`+`を押してタスクを追加し、`Commands`の欄にGit Bashの場所を書けばOK。
このとき、引数に`--login -i`を渡すのと、パス全体をダブルクォートで囲むことに注意する。
![](/images/2015-08-16-vagrant-digitalocean-on-windows/gitbash.PNG)

そのままだと日本語が化けるので、`vi ~/.bash_profile`で以下のエイリアスを追加する。

    alias ls='ls --show-control-chars'

あと、Git Bashにはrsyncが入っていないので、[rsyncをGit for Windowsに混ぜる](http://hail2u.net/blog/software/install-rsync-to-git-for-windows.html)を参考にMinGWのパッケージからexeとdllを持ってくる。
`rsync --version`でバージョン情報が見られればOK。

vagrant-digitaloceanのインストール
----

インストールはコマンド一発。

    vagrant plugin install vagrant-digitalocean

Git Bash(=MSYS)環境の場合、インストールしただけだと`vagrant up`の途中でエラーが出て失敗する。
これは、rsyncでDropletとの同期を設定する時のディレクトリ指定がCygwinスタイル(`/cygdrive/c/Users/...`)になっているため。
ここを設定するオプションは(たぶん)ないので、ソースそのものを書き換えてMSYSスタイル(`/c/Users/...`)になるようにする。

手順としては、同期を設定する処理が書かれているファイルを開き、

    cd ~/.vagrant.d/gems/gems/vagrant-digitalocean-0.7.7/lib/vagrant-digitalocean/actions
    vi sync_folders.rb

37～40行目、ディレクトリ表記を変換している部分から、

    # on windows rsync.exe requires cygdrive-style paths
    if Vagrant::Util::Platform.windows?
      hostpath = hostpath.gsub(/^(\w):/) { "/cygdrive/#{$1}" }
    end

`/cygdrive`を消す。

    # on windows rsync.exe requires cygdrive-style paths
    if Vagrant::Util::Platform.windows?
      hostpath = hostpath.gsub(/^(\w):/) { "/#{$1}" }
    end


Vagrantfileをつくる
----

必要なもの

* Digital OceanのAPIトークン
    + Digital Oceanにログインして、上の「API」メニューからPersonal Access Tokenを発行できる
    + たぶんWRITE権限が必要
    + 発行完了画面を閉じるとトークンそのものは確認できなくなるので、きちんとメモったことを確認してから閉じる
* SSH用の秘密鍵
    + ない場合は`ssh-keygen -t rsa`で生成できる

Vagrant用のディレクトリをつくって、そこにVagrantfileを置く。

    mkdir vagrant
    cd ./vagrant
    vi Vagrantfile

中身は[サンプル](https://github.com/smdahlen/vagrant-digitalocean)を参考にする。
変えるのは秘密鍵の場所とAPIトークンくらい。ディストリビューションとリージョンはお好みで。

    Vagrant.configure('2') do |config|

      config.vm.provider :digital_ocean do |provider, override|
        override.ssh.private_key_path = '秘密鍵へのパス'
        override.vm.box = 'digital_ocean'
        override.vm.box_url = "https://github.com/smdahlen/vagrant-digitalocean/raw/master/box/digital_ocean.box"

        provider.token = 'APIトークン'
        provider.image = 'centos-7-0-x64'
        provider.region = 'sgp1'
        provider.size = '512mb'
      end
    end


起動
----

    vagrant up --provider=digital_ocean

SSHで入ってみる

    vagrant ssh

停止と削除
----

    vagrant halt
    vagrant destroy

destroy時に「本当に削除してもいいですか？」と聞いてくる。
