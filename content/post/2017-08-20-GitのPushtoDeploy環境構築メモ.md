---
Title: "Git の Push to Deploy 環境構築メモ"
Date: '2017-08-20'
slug: "git-push-to-deploy"
categories:
  - "さくらVPS"
  - "CentOS"
  - "git"
---

[Hugo](https://gohugo.io/) でサイトを生成してラクしてるとはいえ、FTPで直上げしてるのも今時どうなんよ、ということで、Git にプッシュしたら自動でビルド＆デプロイする感じにするために、いろいろ手を入れてみたときの作業メモです。

## 1. Hugo のインストール

[公式 GitHub の release](https://github.com/gohugoio/hugo/releases) からダウンロードできます。

Tarball のダウンロードと解凍  
`wget https://github.com/gohugoio/hugo/releases/download/v0.26/hugo_0.26_Linux-64bit.tar.gz`  
`tar xzvf hugo_0.26_Linux-64bit.tar.gz`  

実行権の追加と移動  
`chmod +x hugo`  
`mv hugo /usr/local/bin/hugo`  

## 2. Gitサーバの導入

Git サーバに必要パッケージを `yum` でインストール

パッケージのインストール  
`yum install git-deamon git-all xinetd`  

xinetd の起動  
`chkconfig xinetd on`  
`/etc/init.d/xinetd start`  

Git Deamon ファイルを xinetd に追加  
`vi /etc/xinetd.d/git-daemon`
```sh
# default: off
# description: The git d&#230;mon allows git repositories to be exported using \
#       the git:// protocol.

service git
{
  disable         = no
  socket_type     = stream
  wait            = no
  user            = nobody
  server          = /usr/libexec/git-core/git-daemon
  server_args     = --base-path=/var/lib/git --export-all --user-path=public_git --syslog --inetd --verbose
  log_on_failure  += USERID
}
```

git-deamon設定を反映  
`/etc/rc.d/init.d/xinetd restart`

gitグループ作成して、SSH接続するユーザーをグループに追加  
`groupadd git`  
`usermod -G wheel,git username`  

これで一通り、Git サーバの設定が完了です。

## 3. リモートリポジトリの作成

リモートのベアリポジトリ（プッシュ先）作成  
`mkdir /home/username/repogitoryName.git`  
`cd /home/username/repogitoryName.git`  
`git init --bare --shared`  

ベアリポジトリからワーキングディレクトリを持つ公開用リポジトリを作成  
`git clone /home/username/repogitoryName.git /home/username/hugo`

ベアリポジトリの hooks/post-receive の設定を追加  
`cd /home/username/repogitoryName.git/hooks`  
`vi post-receive`  
```sh
#/bin/bash

# 公開用リポジトリに移動
cd ~/hugo
# ベアリポジトリからコミットをプル
git --git-dir=.git pull ~/repogitoryName.git master
# Hugo をビルド
hugo
```

hooks に実行権限を追加  
`chmod +x post-receive`  


Hugo のビルド先をワーキングディレクトリとは別のところに出力するようオプションを変更。  
例えば、ワーキングディレクトリと同階層にWebサーバの公開領域がある場合、以下のように指定します。  
`publishDir = "../public_html"`

これで Git へのプッシュを検知してがビルドを自動実行して公開領域へデプロイする環境ができました。

## 参考サイト
- http://hachinobu.hateblo.jp/entry/20130731/1375276390
- http://vdeep.net/git-push-deploy#i-3
