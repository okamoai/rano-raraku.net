---
Title: "Amber の Sublime Text 用 Syntax Highlight 作った"
Date: '2015-12-29'
slug: "amber-sytax-highlight"
categories:
  - "Go"
  - "Hugo"
  - "Sublime Text"
---

## 構文ファイル

[GitHub](https://github.com/okamoai/sublime-amber) に公開しました。

## 使い方

Windows は  
`%APPDATA%\Sublime Text 3\Packages\User`  
MacOSX は  
`~/Library/Application\ Support/Sublime\ Text\ 3/Packages/User`  
以下に amber.tmLanguage ファイルを置いて Sublime Text を再起動すると反映されます。  
Sublime Text 2 では未検証ですが、たぶん動くと思います。

## 作ったいきさつ（蛇足）

せっかく立ち上げるサイト、どうせなら WordPress とかではなく、今までに手を付けていないツールを使おう、ということでいろいろ調べてみた結果、フロントエンド界隈で流行ってるっぽいHugoを使うことにしました。  
しかし、標準のGoテンプレート記法はHTMLへのインライン埋め込みで、見た目にあまりスマートじゃありません。  
また、今までJadeでHTMLを書いていたこともあり、「タグいちいち閉じるのメンドくさい」「extends使ってレイアウトをテンプレートファイル単位で管理したい」という気持ちになるのにさほど時間はかかりませんでした。

[公式サイト](https://gohugo.io/)には、Hugo ではテンプレートエンジンとして [Amber](https://github.com/eknkc/amber) と [Ace](https://github.com/yosssi/ace) をサポートしているとありました。
どちらも Jade の機能をベースに開発されているようですが、Amber の方がより Jade の機能に近く、GitHub 上の Star 数もが上だったので、 こちらを採用。

喜び勇んで使ってみたものの、当然のことながら Syntax Highlight 定義がないのでテンプレートを弄るモチベーションが上がりません…
Package Control や ネットで調べてみたところ、まだ Amber の Syntax Highlight は無く、 [公式の Issues](https://github.com/eknkc/amber/issues/13) でも「Jade の定義使ってるわー」な反応だったので、勉強がてら勢いで作ってみました。

最低限の定義なので、インラインの JavaScript や CSS までは対応してません。  
自分がテンプレートをカスタマイズしていくうちに少しずつ調整していこうかと思います。