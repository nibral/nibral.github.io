---
layout: default
title: ffmpeg on CentOS
permalink: ffmpeg-on-centos
---

ffmpeg on CentOS
====

はじめに
----

[公式のインストールガイド](https://trac.ffmpeg.org/wiki/CompilationGuide/Centos)を参考に、余計なオプション入れてないffmpegをビルドしよう(x264とfdk_aac)


Git
----

yumのパッケージが古いので自分でビルドする

git 依存関係の解決

    sudo yum install curl-devel expat-devel gcc gettext-devel make openssl-devel zlib-devel perl-ExtUtils-MakeMaker

Gitのダウンロード

    wget https://www.kernel.org/pub/software/scm/git/git-x.x.x.tar.gz


展開してビルド、インストール

    tar -zxf git-x.x.x.tar.gz
    cd git-x.x.x
    make prefix=/usr/local all
    sudo make prefix=/usr/local install


ffmpeg 依存関係の解決
----

    sudo yum install autoconf automake cmake freetype-devel gcc-c++ libtool nasm pkgconfig

変更点

* gcc,make,zlib-devel…gitのビルドでインストール済
* git…上でビルドしたものと競合
* mercurial…libx265をダウンロードしないので不要

ビルド
----

ダウンロード用のディレクトリを切る

    mkdir ffmpeg-source

yasm

    cd ~/ffmpeg_sources
    git clone --depth 1 git://github.com/yasm/yasm.git
    cd yasm
    autoreconf -fiv
    ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin"
    make
    sudo make install
    make distclean

x264

    cd ~/ffmpeg_sources
    git clone --depth 1 git://git.videolan.org/x264
    cd x264
    ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-static
    make
    sudo make install
    make distclean

fdk_aac

    cd ~/ffmpeg_sources
    git clone --depth 1 git://git.code.sf.net/p/opencore-amr/fdk-aac
    cd fdk-aac
    autoreconf -fiv
    ./configure --prefix="$HOME/ffmpeg_build" --disable-shared
    make
    sudo make install
    make distclean

ffmpegビルド設定

    PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" \
    ./configure --prefix="$HOME/ffmpeg_build" \
    --extra-cflags="-I$HOME/ffmpeg_build/include" \
    --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
    --bindir="$HOME/bin" \
    --pkg-config-flags="--static" \
    --enable-gpl \
    --enable-nonfree \
    --enable-libfdk-aac \
    --enable-libx264

ビルド

    make
    make install
    make distclean

shellのハッシュ更新

    hash -r
