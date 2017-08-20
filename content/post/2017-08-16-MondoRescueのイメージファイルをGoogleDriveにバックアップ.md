---
Title: "Mondo Rescue で Google Drive に定期バックアップ 環境構築メモ"
Date: '2017-08-16'
slug: "mondo-rescue-backup-google-drive"
categories:
  - "さくらVPS"
  - "CentOS"
  - "Mondo Rescue"
  - "Google Drive"
---

サーバ自体はあまりクリティカルなものが動いてないので、いまだにサーバのバックアップ対策をしていなかったんですが、トラブルがあったときに同じ手順繰り返すのもシンドいので、今更ながらにバックアップ対策を実施しました。その時の作業メモです。

## 1. Mondo Rescue の導入

サーバのフルバックアップに [Mondo Rescue](http://www.mondorescue.org/) を使います。  

mondorescue リポジトリを `yum` に登録  
`cd /etc/yum.repos.d`  
`wget ftp://ftp.mondorescue.org/rhel/6/x86_64/mondorescue.repo`  

Mondo Rescue をインストール  
`yum install mondo`

## 2. gdrive の導入

サーバからGoogle Driveへのアクセスには [gdrive](https://github.com/prasmussen/gdrive) を使います。

gdrive のインストール  
`wget -O drive https://docs.google.com/uc?id=0B3X9GlR6EmbnQ0FtZmJJUXEyRTA&export=download`  
`mv drive /usr/sbin/drive`  
`chmod 755 /usr/sbin/drive`

初回起動時に OAth2 認証を行う  
`drive`
```
Go to the following link in your browser:
https://accounts.google.com/o/oauth2/auth?client_id=......
```
上記URLにアクセスし、認証コードを表示し、続いてコンソール上に入力すると使えるようになります。  
`Enter verification code: ***`  


## 3. バックアップ用のシェルスクリプトを作成

`vi /root/server-backup.sh`
```sh
#/bin/bash

readonly BACKUP_DIR=/tmp/backup
readonly BACKUP_BASENAME=vps-backup
readonly PASSWORD=*******
readonly KEEP_DAYS=4

GDRIVE_DIR_ID=$(/usr/sbin/drive list --noheader --query "mimeType='application/vnd.google-apps.folder' and title='$BACKUP_BASENAME'" | awk '{print $1}')

# ディレクトリ初期化
if [ -e $BACKUP_DIR ]
then
  rm -rf $BACKUP_DIR
fi
mkdir $BACKUP_DIR

# サーババックアップ
/usr/sbin/mondoarchive -O -i -N -d $BACKUP_DIR -s 30g -p $BACKUP_BASENAME

# バックアップファイルを暗号化
/usr/bin/openssl aes-256-cbc -e -in ${BACKUP_DIR}/${BACKUP_BASENAME}-1.iso -out ${BACKUP_DIR}/${BACKUP_BASENAME}-crypt.iso -pass pass:$PASSWORD

# Google Drive に保存
/usr/sbin/drive upload --parent $GDRIVE_DIR_ID --file ${BACKUP_DIR}/${BACKUP_BASENAME}-crypt.iso --title ${BACKUP_BASENAME}-`date +%Y%m%d`.iso

# Google Drive の古いバックアップファイルを削除
LIMIT_TIMESTAMP=$(date -d "$KEEP_DAYS days ago" +%s)
/usr/sbin/drive list --noheader --query "'$GDRIVE_DIR_ID' in parents" | while read ln
do
  ITR_ID=$(echo $ln | awk '{print $1}')
  ITR_DATE=$(echo $ln | awk '{print $(NF-1),$NF}')
  if [ $(date -d "$ITR_DATE" +%s) -lt $LIMIT_TIMESTAMP ]
  then
    /usr/sbin/drive delete --id $ITR_ID
  fi
done
```

## cron でシェルを定期実行

`vi /etc/cron.d/backup`
```sh
0 7 * * * root /root/server-backup.sh
```


## 参考サイト
- https://www.riscascape.net/archives/6493
- http://qiita.com/aviscaerulea/items/53123ce5b79c80e31a71
