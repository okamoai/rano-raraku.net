---
Title: "[Qiita] Windows の Node.js 開発環境構築 最小手順"
Date: '2016-05-19'
slug: "win-nodejs-enviroment"
categories:
  - "Node.js"
---
Node.js をの環境構築は Mac 環境だと割りとすんなり行くのですが、Windows 環境では `npm install` を行うとパッケージよってはビルドエラーが出まくって心がヘシ折れそうになります。

エラーが出ないようにするには事前の関連アプリケーションのインストールや環境設定が必要がありますが、ググってみると Visual Studio 2013 Express を入れろとか、いや、Visual Studio 2015 Community じゃないとダメだとか、64bit OS は Windows SDK いれろとか、npm のバージョンを上げろとか、いろんな方法が点在していて、必要最低限の設定がよく分からない状態でした。

そんな放浪の旅を続けていたら、いつの間にか Microsoft の公式ドキュメントができてました。
[Microsoft + Node.js Guidelines - Configuring your Windows development environment
](
https://github.com/Microsoft/nodejs-guidelines/blob/master/windows-environment.md#compiling-native-addon-modules)

ちょうど社内向けに gulp.js を利用した開発環境の構築手順をマニュアル化する必要があったのでここにも記録残していきます。（ほとんど公式ドキュメントの写しですが…）
Windows7 と Windows10 の仮想環境上で動作確認済みです。



# 1. Git のインストール
[公式サイト](https://git-scm.com/download/win) より最新版をダウンロード、インストールします。

- インストール前にコマンドプロンプトで `git --version` と入力してバージョンが返ってきたらその環境にはインストール済みです。インストール済みの環境ではこの項目はスキップしてください。
- インストール時にはデフォルト設定で良いです。"Use Git from the Windows Command Prompt" は必ず有効にしましょう。

# 2. Node.js のインストール
[公式サイト](https://nodejs.org/en/) よりLTS版（Ver.4.4.4 2016年5月19日現在）をダウンロード、インストールします。

- インストール前にコマンドプロンプトで `node -v` や `npm -v` と入力してバージョンが返ってきたらその環境にはインストール済みです。インストール済みの環境ではこの項目はスキップしてください。
- インストールはデフォルト設定でOKです。

# 3. .NET Framework 4.5.1 のインストール（Windows7 のみ）
 [公式サイト](http://www.microsoft.com/en-us/download/details.aspx?id=40773) よりダウンロード、インストールします。

- Windows7 以外の環境はこの項目はスキップしてください。

# 4. Visual C++ Build Tools のインストール
 [公式サイト](http://go.microsoft.com/fwlink/?LinkId=691126) よりダウンロード、インストールします。

- インストールはデフォルト設定でOKです。

# 5. Python のインストール
[公式サイト](https://www.python.org/downloads/) より 2.7 の最新版をダウンロード、インストールします。

- インストール前にコマンドプロンプトで `python --version` と入力して 2.7.x のバージョンが返ってきたらその環境にはインストール済みです。インストール済みの環境ではこの項目はスキップしてください。
- 3.x の最新版ではなく、2.7.x をインストールします。Node.js が参照するライブラリが 2.7 を前提としているためです。
- インストール時に "Add python.exe to Path" を有効にしてパスを通しておきましょう。

# 6. npm 標準オプションの変更
以下内容をコマンドプロンプトで入力します。

```sh
npm config set python python2.7
npm config set msvs_version 2015
```

以上で開発環境の事前準備は完了です。
エラーの出ない`npm install` をお楽しみください！

※この記事は[Qiita](http://qiita.com/okamoai/items/a875b26abab7f18da7d1)とのマルチポストになります。