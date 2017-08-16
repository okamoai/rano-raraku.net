---
Title: "[Qiita] gulp & browserify で 汎用的に 複数のエントリーポイントを管理する"
Date: '2016-11-03'
slug: "browserify-gulp-task"
categories:
  - "gulp"
  - "browserify"
---

Web制作に gulp を使っていと、JavaScript の 連結処理には [gulp-concat](https://www.npmjs.com/package/gulp-concat) が王道ですが、最近は専ら [browserify](http://browserify.org/) を使うようになりました。  
browserify とは node.js で使われる `require` メソッドをブラウザでも使えるようにするツールです。require されたファイルは最終的に一つのファイルに出力されるので、gulp-concat の代替としても使えます。

単に結合するだけなら gulp-concat でも十分なのですが、browserify を使う方が様々なメリットを享受できるので、特に理由がなければ browserify の方をおススメします。  
そこで gulp-concat と比べて browserify を使うと何が嬉しいのか、まとめてみました。

## 1. 依存関係をJavaScript上の文脈で記述できる

gulp-concat を使っていると、gulp タスク側で結合処理を記述する際に依存関係のある JavaScript を結合する場合、

```js:gulpfile.js
var gulp   = require('gulp');
var concat = require('gulp-concat');

gulp.src(['./src/js/jquery.js', './src/js/script.js'])
  .pipe(concat('bundle.js'))
  .pipe(gulp.dest('./dist/'))
;
```

上記のように jquery.js に依存している script.js は jquery.js の後になるよう、結合順に気を遣う必要がありました。  
しかし、gulp タスクはできる限り汎用性を持たせていろいろなプロジェクトに転用したいので、細かな依存関係は、タスクではなく JavaScript 側に記述できた方が、タスクの再利用性が高まり都合が良いです。  
そこで、browserify です。`require` を使って JavaScript の文脈の中で依存関係を記述できるので、gulp タスクから依存関係の記述を追い出すことができます。

```js:gulpfile.js
var gulp       = require('gulp');
var browserify = require('browserify');
var source     = require('vinyl-source-stream');
var buffer     = require('vinyl-buffer');

browserify({ entries: './src/js/bundle.js' })
  .bundle()
  .pipe(source('bundle.js'))
  .pipe(buffer())
  .pipe(gulp.dest('./dist/js/'))
;
```
エントリーポイントとなる./src/js/bundle.jsの中身の例です。

```js:bundle.js
var jQuery = require('jquery');

(function($){
  $(function(){
    console.log('bundle.js Done!');
  });
})(jQuery);
```

## 2. グローバル汚染を最小限に抑えられる

`require` された処理は browserify が生成する即時関数内で完結するため、グローバル変数を汚染するリスクを減らすことができます。  
また、jQuery プラグインなどのため、敢えてグローバル変数に登録したければ、

```js
global.jQuery = require('jquery');
require('jquery.easing');
```
のような形で外出しすることもできます。

## 3. npm で JavaScript パッケージの管理ができる

`npm install` した JavaScript ライブラリを `require` で直接呼ぶことができます。  
こうなると [Bower](https://bower.io/) などのフロントパッケージマネージャーがほぼ不要になり、npm にパッケージ管理を集約できます。

## 4. debug オプションで sourcemap を出力できる

gulp でソースマップを作る際には、gulp-sourcemaps を利用しますが、  
以下のように `debug` オプションを追加すると browserify が soucemap を出力してくれます。

```js
browserify({ entries: './src/js/bundle.js', debug: true })
```

開発環境では debugオプションを `true` にし、本番環境では `false` にすることで、タスク側で行っていた gulp-sourcemap の処理を browserify のオプション一つで適用できます。


## browserify のエントリーポイントを JavaScript ファイル側で管理する

browserify は最終的に一つの JavaScript ファイルに結合しますが、現行サイトの互換性を考慮したりすると、複数のファイルで管理したいプロジェクトもあります。 しかし browserify は複数のエントリーポイントを出力するような仕様はありません。  
汎用的に使う gulp タスクを目指すのであれば、タスクにエントリーポイントを指定するような仕組みを避け、JavaScript のファイル構成でエントリーポイントが決められるようにするとタスクの再利用性が高まります。  
複数のエントリーポイントを一度に扱えない弱点（仕様）を gulp のタスクでカバーします。

### JavaScript のファイル構成例

ファイル出力に Pug や Sass でおなじみのルールを使います。ファイル名の先頭にアンダースコア（_）が付いたファイルは出力の対象外とし、アンダースコアなしのファイルがエントリーポイントとなるルールとします。

エントリーポイントの JavaScript ファイルでは npm パッケージのライブラリやアンダースコア付き JavaScript ファイルを `require` する記述で構成されます。

```js:lib.js
global.jQuery = require('jquery'); // npm パッケージから呼び出し
require('jquery.easing'); // npm パッケージから呼び出し
```
```js:app.js
require('./_console1.js'); // ローカルファイルから呼び出し
require('./_console2.js'); // ローカルファイルから呼び出し
```
```js:_console1.js
(function($){
  $(function(){
    console.log('console1 Done.');
  });
})(jQuery);
```
```js:_console2.js
(function($){
  $(function(){
    console.log('console2 Done.');
  });
})(jQuery);
```

このルールであれば、エントリーポイントを追加・変更をすることも JavaScript ファイルでコントロールできるため、gulp タスク側でエントリーポイントを指定する必要がなくなり、タスクの再利用性が高まります。

### 実際の gulp タスク

前述のルールを適用し、実現するための gulp タスクが以下になります。

```js:gulpfile.js
'use strict';

var glob       = require('glob');
var gulp       = require('gulp');
var gulpIf     = require('gulp-if');
var uglify     = require('gulp-uglify');
var plumber    = require('gulp-plumber');
var notify     = require('gulp-notify');
var browserify = require('browserify');
var source     = require('vinyl-source-stream');
var buffer     = require('vinyl-buffer');
var minimist   = require('minimist');

// タスクの引数（環境情報）を取得
var options = minimist(process.argv.slice(2), {
  string:  ['env'],
  default: {
    env: 'development'
  }
});

gulp.task('js', function(callback){

  // JavaScriptファイルのからエントリーポイントのファイルを抽出
  var jsFiles = glob.sync( './src/js/{!(_)*.js,**/!(_)*/!(_)*.js}' );
  if (jsFiles.length === 0) {
    callback();
  }

  // タスクの終了を通知
  var task_num = jsFiles.length;
  var task_executed = 0;
  var onEnd = function () {
    if (task_num === ++task_executed) {
      callback();
    }
  };

  // browserify で結合
  jsFiles.forEach(function(file) {
    var fileName = file.replace(/.+\/(.+\.js)/, '$1');
    var filePath = file.replace(new RegExp('./src/(.*)/.+\.js'), '$1');
    browserify({
      entries: file,
      debug: options.env === 'development' ? true : false
    })
      .bundle()
      .on('end', onEnd)
      .pipe(source(fileName))
      .pipe(buffer())
      .pipe(plumber({
        errorHandler: notify.onError('<%= error.message %>')
       }))
      .pipe(gulpIf(
        options.env === 'staging' || options.env === 'production',
        uglify({ preserveComments: 'some' })
      ))
      .pipe(gulp.dest('./dist/'+filePath))
    ;
  });

});
```

## データダウンロード
GitHub に上記のデータ一式を公開しました。  
使い方は README を参照してください。  
[gulp task example: Browserify multi entry point](https://github.com/okamoai/example-gulp-task-browserify-multi-entry-point)

※この記事は[Qiita](http://qiita.com/okamoai/items/033a865c997540c6dcf7)とのマルチポストになります。