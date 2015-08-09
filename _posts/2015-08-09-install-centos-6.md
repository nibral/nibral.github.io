---
layout: default
title: CentOS 6インストールと初期設定
permalink: install-centos-6
---

CentOS 6インストールと初期設定
====

netinstallが終わって、rootでログインしたところから

パッケージのアップデート
---

    yum update

suの設定変更
----

suをwheelグループに限定

    vi /etc/pam.d/su
        -# auth required pam_wheel.so use_uid
        +auth required pam_wheel.so use_uid

sudoの設定

    visudo
        -# %wheel ALL=(ALL) ALL
        +%wheel ALL=(ALL) ALL

作業用ユーザ作成
----

ユーザ追加

    useradd -G wheel akari

パスワード変更

    passwd akari

SSHの設定
----

**SSHクライアントを使って、作業用ユーザでログイン(22番ポート)**

ssh鍵作成

    ssh-keygen
    cp .ssh/id_rsa.pub .ssh/authorized_keys
    chmod 600 .ssh/authorized_keys

手元の端末に秘密鍵を持ってくる

    cat .ssh/id_rsa

**作業用ユーザからログアウト、rootに戻る**

sshdの設定変更(ポート変更、rootログイン無効、鍵認証を強制)

    vi /etc/ssh/sshd_config
        Port 10022
        PermitRootLogin no
        PasswordAuthenication no
        AuthorizedKeysFile .ssh/authorized_keys


ファイアウォールの設定
---

必要パッケージのインストール

    yum install dbus dbus-python
    chkconfig messagebus on
    /etc/init.d/messagebus start


ファイアウォール設定ツールインストール

    yum install system-config-firewall-tui

起動

    system-config-firewall-tui
        Customizeを選び、必要なポートを開ける
        終わったらClose→OK

SSHに印をつけて22番ポートを開けると攻撃される可能性があるので、
"Other Ports"で別のポートをSSH用に開けるとよい(10022番とか)

**生成される`/etc/sysconfig/iptables`の例**(http,https,10022)

    # Firewall configuration written by system-config-firewall
    # Manual customization of this file is not recommended.
    ＊filter
    :INPUT ACCEPT [0:0]
    :FORWARD ACCEPT [0:0]
    :OUTPUT ACCEPT [0:0]
    -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
    -A INPUT -p icmp -j ACCEPT
    -A INPUT -i lo -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 10022 -j ACCEPT
    -A INPUT -j REJECT --reject-with icmp-host-prohibited
    -A FORWARD -j REJECT --reject-with icmp-host-prohibited
    COMMIT

SELinuxを切る
----

SELinuxが入っていると制限がきついので切る(sshのポートを変えたときに通信できないとか)

    vi /etc/selinux/config
        SELINUX=disabled

全部おわったら再起動
----

    reboot now
