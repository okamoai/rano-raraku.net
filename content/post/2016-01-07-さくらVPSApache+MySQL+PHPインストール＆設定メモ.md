---
draft: false
Title: "さくらVPS Apache+MySQL+PHP インストール＆設定メモ"
Date: '2016-01-07'
slug: "sakura-vps-web-db-setup"
categories:
  - "さくらVPS"
  - "CentOS"
  - "Apache"
  - "MySQL"
  - "PHP"
---

さくら VPS 契約後のほぼ自分用セットアップメモ第二弾です。  
過去のメモの整理がてらのまとめです。  
今後、他にもサーバに手を加えた時にはメモを落としていこうかと思います。

## 1. 標準レポジトリはバージョンが古いので Remi リポジトリを追加

### 1-1. まずは EPEL リポジトリ を yum で追加  
`yum -y install epel-release`

yum コマンド時に明示的に EPEL リポジトリを参照するように設定を変更  
`vi /etc/yum.repos.d/epel.repo`
```diff
- enabled=1
+ enabled=0
```

実際に yum で EPEL リポジトリを指定するときは以下のようにオプションを追加する  
`yum --enablerepo=epel list`

### 1-2. 続いて Remi リポジトリを追加

設定パッケージをダウンロード  
`wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm`
インストール  
`rpm -Uvh remi-release-6.rpm`

実際に yum で Remi リポジトリを指定するときは以下のようにオプションを追加する  
`yum --enablerepo=remi list`

## 2. Apache のインストール

### 2−1. yum 経由で Apache のインストール
`yum --enablerepo=remi -y install httpd`

### 2−2. 設定ファイルを更新
`vim /etc/httpd/conf/httpd.conf`

44行目 サーバ応答ヘッダを制限
```diff
- ServerTokens OS
+ ServerTokens Prod
```
262行目 サーバ管理者連絡先を変更
```diff
- ServerAdmin root@localhost
+ ServerAdmin example@example.com
```

276行目 サーバホスト名を変更
```diff
- #ServerName www.example.com:80
+ ServerName www15261uj.sakura.ne.jp:80
```

303行目 ディレクトリ一覧を非表示
```diff
- Options FollowSymLinks
+ Options -Indexes FollowSymLinks
```

551-558行目 icons 設定をコメントアウト

536行目 サーバ情報を非表示
```diff
- ServerSignature On
+ ServerSignature Off
```

855行目 error エイリアスをコメントアウト
```diff
- Alias /error/ "/var/www/error/“
+ #Alias /error/ "/var/www/error/"
```

990行目 VirtualHost を有効にする
```diff
-  #NameVirtualHost *:80
+ NameVirtualHost *:80
```

最後の行にバーチャルホスト用の設定を追加

```conf
<virtualHost *:80>
     ServerName [ドメイン名]
     DocumentRoot /home/[ユーザー名]/public_html
     CustomLog /home/[ユーザー名]/logs/access_log common
     ErrorLog /home/[ユーザー名]/logs/error_log
     <Directory />
          AllowOverride All
     </Directory>
</virtualHost>
```

### 2−3. 関連ディレクトリを作成  
`mkdir -p /home/[ユーザー名]/{public_html,logs}`

### 2−4. Apacheサービスの設定
設定ファイルの構文チェック  
`apachectl configtest`

Apache起動  
`apachectl start`

Apache の自動起動設定  
`chkconfig httpd on`

自動起動設定の反映を確認  
`chkconfig --list httpd`

### 2−5. Apache のログを logrotate でローテーション

`vim /etc/logrotate.d/httpd`

最終行に以下を追加（1ヶ月ごと1年分でローテーション）
```conf
/home/[ユーザー名]/logs/*log {
     create
     missingok
     notifempty
     monthly
     rotate 12
     compress
     sharedscripts
     postrotate
          /sbin/service httpd reload > /dev/null 2>/dev/null || true
     endscript
}
```

## 3. MySQL のインストール

### 3−1. yum で MySQL をインストール
`yum install --enablerepo=remi -y mysql-server`

### 3−2. MySQL サービスの設定

MySQL の自動起動設定  
`chkconfig mysqld on`

自動起動設定の反映を確認  
`chkconfig --list mysqld`

サービス開始  
`/etc/init.d/mysqld start`

rootのパスワードを変更  
`mysqladmin -u root password ***`

設定ファイルを変更、最後の行に追加（日本語対応）  
`vim /etc/my.cnf`

```ini
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqldump]
default-character-set=utf8
```

## 4. PHPのインストール

### 4−1. yum で GD ライブラリの最新版を先にインストール

最新版の GD でないと remi-php56 レポジトリで PHP のインストールがでエラーになるため  
`yum -y --enablerepo=remi install gd-last`

### 4−2. yum で mcrypt ライブラリの最新版を先にインストール
最新版の mcrypt でないと remi-php56 レポジトリで PHP のインストールがでエラーになるため  
`yum --enablerepo=epel -y install libmcrypt libmcrypt-devel`

### 4−3. yum で PHP と関連ライブラリをインストール
`yum --enablerepo=remi-php56 install php php-devel php-mbstring php-mysql php-pdo php-mcrypt php-pear php-xml  php-gd php-pecl-apc`

### 4−4. PHP の設定
`vim /etc/php.ini`

366行目 PHP 情報を非表示
```diff
- expose_php = On
+ expose_php = Off
```

575行目 PHP のエラー出力先を変更
```diff
- ;error_log = php_errors.log
+ error_log = /var/log/php_errors.log
```

889行目 タイムゾーンを東京に変更
```diff
-;date.timezone =
+date.timezone = Asia/Tokyo
```

1660行目 言語設定を日本語に変更
```diff
-;mbstring.language = Japanese
+mbstring.language = Japanese
```

1667行目 内部エンコーディングを UTF-8 に変更
```diff
-;mbstring.internal_encoding =
+mbstring.internal_encoding = UTF-8
```

1675行目 入力エンコードを自動に変更
```diff
-;mbstring.http_input =
+mbstring.http_input = auto
```

1698行目 エンコード検知順を自動に変更
```diff
-;mbstring.detect_order = auto
+mbstring.detect_order = auto
```

### 4−5. PHP の APC を有効にする
APC 用の設定ファイルを作成  
`vim /etc/php.d/apc.ini`

```ini
extension = apc.so
apc.enabled = 1
apc.shm_size = 128M
apc.ttl = 3600
apc.user_ttl = 3600
apc.gc_ttl = 7200
apc.stat = 1
```

Apacheを再起動  
`service httpd restart`

## 参考サイト

- http://blog.orangemittoo.net/post/sakuravps_init2/
- [http://akabeko.me/blog/2010/09/さくらのVPS-を使いはじめる-4](http://akabeko.me/blog/2010/09/%E3%81%95%E3%81%8F%E3%82%89%E3%81%AEvps-%E3%82%92%E4%BD%BF%E3%81%84%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B-4/)
- http://akabeko.me/blog/2012/04/revps-04-apache/
- http://akabeko.me/blog/2012/04/revps-05-mysql/
- http://blog.nzakr.com/vps-lamp/