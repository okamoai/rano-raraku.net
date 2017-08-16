---
Title: "[Qiita] CSS 設計における Modifier の記述ルールの最適化"
Date: '2016-02-14'
slug: "css-rules-modifier"
categories:
  - "CSS"
---
みなさんは CSS 設計をするとき、どの設計方針を採用してますか？
自分も [SMACSS](https://smacss.com/)、[BEM](http://getbem.com/)、[FLOCSS](https://github.com/hiloki/flocss) と渡り歩いて来ましたが、どうにもしっくり来ない点が Modifier の記述ルールです。
ここでは自分の試行錯誤の過程と結果を公開してみました。

## BEM 記法

クラス名に構造情報を持たせることで、要素のモジュール化を強要して定義の破綻を防ぐ、シンプルかつ非常に強力なルールなのですが、下記の例のように HTML 側のクラス記述が冗長になるのがデメリットです。  

```css:CSS
.local-menu { … } /* Block */
  .local-menu--category { … }  /* Modifier */
  .local-menu__title { … }  /* Element */
  .local-menu__list { … }  /* Element */
    .local-menu__list--category { … }  /* Modifier */
    .local-menu__list__item { … }  /* Element */
      .local-menu__list__item--typeA { … }  /* Modifier */
      .local-menu__list__item--typeB { … }  /* Modifier */
      .local-menu__list__item--category { … }  /* Modifier */
      .local-menu__list__item--active { … }  /* Modifier */
```
```html:HTML
<nav class="local-menu">
  <h2 class="local-menu__title">メニュー見出し（Basis）</h2>
  <ul class="local-menu__list">
    <li class="local-menu__list__item local-menu__list__item--typeA local-menu__list__item--active"><a href="#">メニューA</a></li>
    <li class="local-menu__list__item local-menu__list__item--typeB"><a href="#">メニューB</a></li>
  </ul>
</nav>

<nav class="local-menu local-menu--category">
  <h2 class="local-menu__title">メニュー見出し（Modifier）</h2>
  <ul class="local-menu__list local-menu__list--category">
    <li class="local-menu__list__item local-menu__list__item--typeA local-menu__list__item--category local-menu__list__item--active"><a href="#">メニューA</a></li>
    <li class="local-menu__list__item local-menu__list__item--typeB local-menu__list__item--category"><a href="#">メニューB</a></li>
  </ul>
</nav>
```

CSS を見ると構成は非常に分かりやすいのですが、ネストが長くなった Element からの Modifier ともなると HTML のクラス記述が一気に膨れ上がってしまい、HTML ソースの class の箇所は何かもう見るもの嫌になってきます。目が滑ってしまい、どうにも CSS の構成が頭に入ってきません。


## SMACSS、FLOCCSS の識別子記法

SMACSS、FLOCCSS ではクラス名に以下のようなプレフィックスを付けて意味を持たせる手法を採用しています。
`.l-*`： レイアウト構造を示すクラス
`.is-*`： 要素の状態を示すクラス
`.c-*`： コンポーネント、モジュールを示すクラス
`.p-*`： プロジェクト固有の要素を示すクラス
`.u-*`： ユーティリティ、ちょっとしたスタイル調整をするクラス
といった意味があります。

こちらの記法で先ほどの記述を書き直してみます。

```css:CSS
.c-local-menu { … } /* Block */
  .c-local-menu--category { … }  /* Modifier */
  .c-local-menu__title { … }  /* Element */
  .c-local-menu__list { … }  /* Element */
    .c-local-menu__list--category { … }  /* Modifier */
    .c-local-menu__list__item { … }  /* Element */
      .c-local-menu__list__item--typeA { … }  /* Modifier */
      .c-local-menu__list__item--typeB { … }  /* Modifier */
      .c-local-menu__list__item--category { … }  /* Modifier */
      .c-local-menu__list__item.is-active { … }  /* State */
```
```html:HTML
<nav class="c-local-menu">
  <h2 class="c-local-menu__title">メニュー見出し（Basis）</h2>
  <ul class="c-local-menu__list">
    <li class="c-local-menu__list__item local-menu__list__item--typeA is-active"><a href="#">メニューA</a></li>
    <li class="c-local-menu__list__item local-menu__list__item--typeB"><a href="#">メニューB</a></li>
  </ul>
</nav>

<nav class="c-local-menu c-local-menu--category">
  <h2 class="c-local-menu__title">メニュー見出し（Modifier）</h2>
  <ul class="c-local-menu__list c-local-menu__list--category">
    <li class="c-local-menu__list__item local-menu__list__item--typeA local-menu__list__item--category is-active"><a href="#">メニューA</a></li>
    <li class="c-local-menu__list__item local-menu__list__item--typeB local-menu__list__item--category"><a href="#">メニューB</a></li>
  </ul>
</nav>
```

現在地表示部分が SMACSS の状態を示すマルチクラス記法になったので、クラス名の重複分がなくなり、ちょっとだけスッキリしました。こうなると、Modifier も全部マルチクラスにしてはどうだろうか、と思い始めてきます。

## 識別子の付いていないクラス名は全て Modifier

ここからが独自記法のルールになります。
FLOCSS ではクラスに識別子を付けて管理しているため、逆を返せば、識別子が付いていないクラスはすべて Modifier というルールにしてしまうことで、マルチクラス記法に統一することができます。

```css:CSS
.c-local-menu { … } /* Block */
  .c-local-menu.category { … }  /* Modifier */
  .c-local-menu__title { … }  /* Element */
  .c-local-menu__list { … }  /* Element */
    .c-local-menu__list.category { … }  /* Modifier */
    .c-local-menu__list__item { … }  /* Element */
      .c-local-menu__list__item.typeA { … }  /* Modifier */
      .c-local-menu__list__item.typeB { … }  /* Modifier */
      .c-local-menu__list__item.category { … }  /* Modifier */
      .c-local-menu__list__item.is-active { … }  /* State */
```
```html:HTML
<nav class="c-local-menu">
  <h2 class="c-local-menu__title">メニュー見出し（Basis）</h2>
  <ul class="c-local-menu__list">
    <li class="c-local-menu__list__item typeA is-active"><a href="#">メニューA</a></li>
    <li class="c-local-menu__list__item typeB"><a href="#">メニューB</a></li>
  </ul>
</nav>

<nav class="c-local-menu category">
  <h2 class="c-local-menu__title">メニュー見出し（Modifier）</h2>
  <ul class="c-local-menu__list category">
    <li class="c-local-menu__list__item typeA category is-active"><a href="#">メニューA</a></li>
    <li class="c-local-menu__list__item typeB category"><a href="#">メニューB</a></li>
  </ul>
</nav>
```

このルールには以下のようなメリットがあります。

- 記述が HTML では `class="c-label-menu category"`、CSS では `.c-label-menu.category` とほぼ同じ構成で記述できるため、両方のコードから関連性を参照しやすい
- 人間がクラス定義を読んでも文脈として意図を汲み取りやすい
- 重複記述が減り、コード量も減少する
- CSS 上では Modifier は必ずマルチクラスで Block や Element とセットで記述するため、別のコンポーネントとクラス名の衝突が発生せず、意図しない上書きが発生しない

## BEM 記法の圧縮

個人的な好き嫌いかもしれませんが、やはり区切り文字が記号2文字は冗長に感じます。
また、FLOCSS の識別子記法でハイフン記号を使っているため、Block に相当するクラスには視認性の点からもあまり記号を使いたくありません。
Element の文字区切りは1文字として、単語の区切りは Lower Camel Case にすることでクラス名の表記に調子がついて視覚的にだいぶ読みやすくなります。

```css:CSS
.c-localMenu { … } /* Block */
  .c-localMenu.category { … }  /* Modifier */
  .c-localMenu_title { … }  /* Element */
  .c-localMenu_list { … }  /* Element */
    .c-localMenu_list.category { … }  /* Modifier */
    .c-localMenu_list_item { … }  /* Element */
      .c-localMenu_list_item.typeA { … }  /* Modifier */
      .c-localMenu_list_item.typeB { … }  /* Modifier */
      .c-localMenu_list_item.category { … }  /* Modifier */
      .c-localMenu_list_item.is-active { … }  /* State */
```
```html:HTML
<nav class="c-localMenu">
  <h2 class="c-localMenu_title">メニュー見出し（Basis）</h2>
  <ul class="c-localMenu_list">
    <li class="c-localMenu_list_item typeA is-active"><a href="#">メニューA</a></li>
    <li class="c-localMenu_list_item typeB"><a href="#">メニューB</a></li>
  </ul>
</nav>

<nav class="c-localMenu category">
  <h2 class="c-localMenu_title">メニュー見出し（Modifier）</h2>
  <ul class="c-localMenu_list category">
    <li class="c-localMenu_list_item typeA category is-active"><a href="#">メニューA</a></li>
    <li class="c-localMenu_list_item typeB category"><a href="#">メニューB</a></li>
  </ul>
</nav>
```

## 子孫セレクタを使用して Modifier の記述を Block 要素のみに集約

BEM 記法では子孫セレクタを使わずにフラットに書いていくため、Modifier による派生スタイルを Element に適用していくのに新たなクラス名を付ける必要がありましたが、構成要素が変わらないのであれば、Modifier は Block にのみ適用した方が HTML 側のクラス定義をさらにシンプルにできます。

```css:CSS
.c-localMenu { … } /* Block */
  .c-localMenu_title { … }  /* Element */
  .c-localMenu_list { … }  /* Element */
    .c-localMenu_list_item { … }  /* Element */
      .c-localMenu_list_item.typeA { … }  /* Modifier */
      .c-localMenu_list_item.typeB { … }  /* Modifier */
      .c-localMenu_list_item.is-active { … }  /* State */
  .c-localMenu.category { … }  /* Modifier */
  .c-localMenu.category .c-localMenu_list { … }  /* Modifier - Element */
  .c-localMenu.category .c-localMenu_list_item { … }  /* Modifier - Element */
```
```html:HTML
<nav class="c-localMenu">
  <h2 class="c-localMenu_title">メニュー見出し（Basis）</h2>
  <ul class="c-localMenu_list">
    <li class="c-localMenu_list_item is-active"><a href="#">メニューA</a></li>
    <li class="c-localMenu_list_item"><a href="#">メニューB</a></li>
  </ul>
</nav>

<nav class="c-localMenu category">
  <h2 class="c-localMenu_title">メニュー見出し（Modifier）</h2>
  <ul class="c-localMenu_list">
    <li class="c-localMenu_list_item is-active"><a href="#">メニューA</a></li>
    <li class="c-localMenu_list_item"><a href="#">メニューB</a></li>
  </ul>
</nav>
```

HTML 上で Element に定義していた Modifier を Block に一本化したため、category の有無でこのコンポーネントの派生を切り替えることができるようになりました。
これは JavaScript やプログラムから条件によって Modifier を付与する必要がある場合、複数の要素に対してクラスを適用しなくて済むというメリットがあります。

## Sass で記述を圧縮

HTML が簡潔になった代わりに CSS の方が子孫セレクタの記述をした分、ずいぶんと冗長な個所が増えてしまいました。
ここで Sass を使って冗長になった記述を圧縮します。

```scss:SCSS
.c-localMenu {
  $c: &;
  &_title { … }
  &_list {
    …
    &_item {
      …
      &.typeA { … }
      &.typeB { … }
      &.is-active { … }
    }
  }
  &.category {
    …
    #{$c}_list {
      …
      &_item { … }
    }
  }
}
```

Block 名を最初の記述のみとし、後続の記述はすべて「&」を使用して参照することで Block 名が何度も登場することを避けています。Block のクラス名を変更すれば自動で関連する Element にすべて反映されるようになります。

また、直下の `$c: &;` は Block のクラス名を格納しています。なぜこのような形を取るかというと、Block の Modifier 以下、子孫セレクタを展開する際に、最初の Block と同じように参照すると、以下の例のようにうまく展開されないためです。

```scss:SCSS
.c-localMenu {
  &_list {
    &_item {
    }
  }
  &.category {
    &_list {
      &_item {
      }
    }
  }
}
/*
.c-localMenu { … }
.c-localMenu_list { … }
.c-localMenu.category { … }
.c-localMenu.category_list { … }
.c-localMenu.category_list_item { … }
と出力され、想定した子孫セレクタが得られない。
.c-localMenu.category .c-localMenu_list_item { … } ←こうなってほしい
*/
```

「&」は直近の親のクラス名を継承するので、Block 名定義直後に変数にクラス名を保存して、Modifier 内の定義で変数を再利用することで、この問題を回避しています。

## Modifier の Modifier を記述する

トップレベルのModifierを記述する分には問題ないのですが、子階層の Element の Modifier からの Element を定義すると Sass 記法がさらにややこしくなります。

例えば、以下のようなHTMLがあり、

```html:HTML
<nav class="c-localMenu category">
  <h2 class="c-localMenu_title">メニュー見出し（Modifier）</h2>
  <ul class="c-localMenu_list">
    <li class="c-localMenu_list_item is-active"><a href="#" class="c-localMenu_list_item_link">メニューA</a></li>
    <li class="c-localMenu_list_item"><a href="#" class="c-localMenu_list_item_link">メニューB</a></li>
  </ul>
</nav>
```
マルチクラスで Modifier -> Element -> Modifier -> Element を参照しようとすると、
`.c-localMenu.category .c-localMenu_list_item.typeA .c-localMenu_list_item_link` のような記述になります。ちょっとこれはグッタリ来ますね…
Sassで書き直すと以下のような形。

```scss:SCSS
.c-localMenu {
  $c: &;
  &_title {
    …
  }
  &_list {
    …
    &_item {
      …
      &.typeA { … }
      &.typeB { … }
      &.is-active { … }
      &_link {
        …
      }
    }
  }
  &.category {
    …
    #{$c}_list {
      …
      &_item {
        …
        &.typeA {
          …
          #{$c}_list_item_link {
            …
          }
        }
        &_link {
          …
        }
      }
    }
  }
}
```
`#{$c}_list_item_link` あたりの表記は Element の二重記述になるので、何とか書かなくてもよい方法を模索しましたが、中間変数や配列を用意しても、結局呼び出し定義が煩雑になるので、あまり良い手立てはありませんでした。何かいい方法があったら教えてください。

## おわり

以上、BEM から始めた記法を無駄のない記法にカスタマイズした結果を晒してみました。
従来の記法に比べて「コード量が少ない」「クラス定義が文脈として読み取れる」「定義が衝突しにくい」が最大のメリットです。

※この記事は[Qiita](http://qiita.com/okamoai/items/1d2c9018a79e4dee69f4)とのマルチポストになります。