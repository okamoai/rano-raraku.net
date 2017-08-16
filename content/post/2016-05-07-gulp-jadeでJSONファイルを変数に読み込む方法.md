---
Title: "[Qiita] gulp-jade で JSON ファイルを変数に読み込む方法"
Date: '2016-05-07'
slug: "jade-json-variable"
categories:
  - "gulp"
  - "jade"
---
jade でサイトを作成していると、共通情報は外部ファイルにまとめて記述したくなります。やはり jade なので、外部ファイルは JSON 形式にまとめておきたいところですね。

# 先に結論
[gulp-data](https://www.npmjs.com/package/gulp-data) を使いましょう。
[gulp-jade](https://www.npmjs.com/package/gulp-jade) に使い方も紹介されてます。

# gulp-data を使わない方法
公式の方法に辿り着く前に、ろくすっぽ調べずに自己流で JSON データをパースしてました…。
果たしてこの方法に需要があるのかどうか分かりませんが、試行錯誤の時間を無為にするのも悔しいので、まあ、こんな方法もあるよーということで、晒しておきます。

<script src="https://gist.github.com/okamoai/221f6513187f4e48052daaac86bdeff9.js"></script>

# 解説
jade 標準出力を横取りして JSON をパースしてます。
最後に読み込んだ JSON が標準出力されないよう、バックアップしていた `buf` を書き戻してます。 `buf` は jade の mixins で使われている内部変数ですので、あまり行儀の良いやり方ではありませんが…

強いてこの方法の利点を挙げれば

- gulpfile.js に 設定ファイルの記述なしに jade の構成ファイルの記述内で完結できる
- gulp-data が何らかの理由により使えない環境の代替となる

と言ったところでしょうか。

※この記事は[Qiita](http://qiita.com/okamoai/items/224098a432e6e6df6da3)とのマルチポストになります。