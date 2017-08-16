---
Title: "[Qiita] gulp-pug v2.0 で JSON ファイルを変数に読み込む方法"
Date: '2016-06-23'
slug: "pug-json-variable"
categories:
  - "gulp"
  - "jade"
  - "pug"
---

[jade](https://github.com/jadejs/jade) は商標の問題から、[pug](https://github.com/pugjs/pug) に名前が変わりました。また、2016年6月23日時点で v2.0 がベータ版リリースされていますが、v1.11から いろいろと内部構成が変わっていて、[以前に紹介した mixin で横取りする方法](/201605/jade-json-variable/)が使えなくなっていました。

素直に [gulp-data](https://www.npmjs.com/package/gulp-data) を使った方が良さそうですが、ページ毎に適用する JSON ファイルを変更するような運用にしたいので、JSON ファイルの指定は gulpfile.js ではなく、pug ファイル側で指定したいところです。
pug 自体に外部 JSON ファイルをパースする機能があれば良いのですが、残念ながら現時点では自前で用意するしかなさそうです。

## タスク側から pug に記述した JSON ファイルを取得する

pug 本体に手を加えるのは時間がかかりそうなので、手っ取り早く gulpfile.js のタスク処理側でカバーします。
ページ側に特定書式コメントの JSON ファイルのパスを指定しておき、タスクの gulp-data 経由で JSON ファイルを読み込むようにします。

pug コメントの後に続けて `data path/to/filename.json` と記述するルールとします。

<script src="https://gist.github.com/okamoai/221f6513187f4e48052daaac86bdeff9.js?file=site.json"></script>

<script src="https://gist.github.com/okamoai/221f6513187f4e48052daaac86bdeff9.js?file=demo.pug"></script>



pug コンパイルのタスクで、前述の JSON パスコメントを検知して、オブジェクトとして取り込むように、以下のように記述します。

<script src="https://gist.github.com/okamoai/221f6513187f4e48052daaac86bdeff9.js?file=gulpfile.js"></script>

これで 指定した JSON ファイルの内容を `data.[ファイル名]` のデータとして pug から参照できるようになりました。

※この記事は[Qiita](http://qiita.com/okamoai/items/224098a432e6e6df6da3)とのマルチポストになります。