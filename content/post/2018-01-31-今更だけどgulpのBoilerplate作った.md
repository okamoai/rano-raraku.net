---
Title: "今更だけど gulp の Boilerplate 作った"
Date: "2018-01-31"
slug: "gulp-boilerplate"
categories:
  - "gulp"
  - "pug"
  - "ejs"
  - "sass"
  - "postcss"
  - "browserify"
  - "制作ツール"
---

実はだいぶ前に作ったあったものの、いろんなことにかまけてるうちにまとめる気力が減って途中で止まっていたのです。  
ちょうどまとまった時間ができたので、乗り掛かった舟、勢いでまとめて公開しておきます。

[https://github.com/okamoai/gulp-boilerplate](https://github.com/okamoai/gulp-boilerplate)

フロントエンド界隈だと gulp 自体が若干レガシー化して来た感あるんですが、まだまだ案件でも使えると思うのでもし良かったら触ってやってみてください。  
最近はタスクランナー使わずに npm scripts でやった方がよさげですが、実はそっちの Boilerplate も作ってあったりします。これもそのうちリポジトリに上げる予定です。

[【追記】2019/6/25: Boilerplate 上げました](../../201906/static-web-boilerplate/)

## Feature

- 静的サイトの出力に適しています。
- 極力 gulp タスクのコードそのものは修正せず、 `gulp/config.js` ファイルと `src/` 以下の開発ファイルのみで既存案件の様々な構成に対応できるようになっています。
- 出力環境によって変わる CDN リソースやドメイン、ドキュメントルートパスなどを `development`, `staging`, `production` 別に定義できます。
- 開発ファイルは以下の形式に対応しています。複数形式のあるものはどちらかを `gulp/config.js` で選択できます。今後、gulp タスクを増やすだけで対応形式を追加できる構成になっています。
  - HTML: [Pug](https://pugjs.org/) or [EJS](http://ejs.co/)
  - CSS: [Sass](http://sass-lang.com/) or [PostCSS](http://postcss.org/)
  - JavaScript: [Babel(ES6)](https://babeljs.io/) or [ES5+ファイル連結 (gulp-concat)](https://github.com/gulp-community/gulp-concat)
  - Web フォント: [SVG から フォント/CSS ファイルを生成 (gulp-iconfont)](https://github.com/nfroidure/gulp-iconfont)
  - CSS スプライト: [PNG から スプライト画像/CSS ファイルを生成 (gulp.spritesmith)](https://github.com/twolfson/gulp.spritesmith)
- 以前に TIPS で書いた以下の内容はタスクの機能として取り込んであります。
  - [gulp-jade で JSON ファイルを変数に読み込む方法](/201605/jade-json-variable/)
  - [gulp で複数のスプライト画像や Web フォントを管理する](/201606/example-gulp-task-split-by-directory/)
  - [gulp & browserify で 汎用的に 複数のエントリーポイントを管理する](/201611/browserify-gulp-task/)
- 詳しい使い方は[リポジトリの README](https://github.com/okamoai/gulp-boilerplate/blob/master/README.md)を見て、実際にビルドしてみるのが早いです。

## GitHub

[https://github.com/okamoai/gulp-boilerplate](https://github.com/okamoai/gulp-boilerplate)

## ToDo

今後想定される作業は以下のような感じです。

- 対応形式の追加（要望があれば）
- ~~バンドラを browserify から webPack に変更~~ [v1.4.0](https://github.com/okamoai/gulp-boilerplate/tree/v1.4.0)で対応しました
- ~~gulp v4 の正式リリース対応~~ [v1.3.0](https://github.com/okamoai/gulp-boilerplate/tree/v1.3.0)で対応しました
- ~~Babel v7 の正式リリース対応~~ [v1.2.0](https://github.com/okamoai/gulp-boilerplate/tree/v1.2.0)で対応しました
