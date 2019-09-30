---
Title: "LitElement を Webpack でバンドルして IE11 で動かすための最適解"
Date: "2019-09-30"
slug: "litelement-with-ie11"
categories:
  - "litElement"
  - "WebComponents"
  - "Webpack"
---

WebComponents の対応状況がだいぶいい感じになってきました。

[WebComponents のブラウザ対応状況 - Can I use](https://caniuse.com/#search=webcomponents)

スマホ環境は iOS で一部 Scoped CSS に制限があるものの、ほぼネイティブに対応していますし、PC 環境は IE/Edge がネックですが、Edge は遠からず Chronium 版に移行する運命。これで Edge で対応が遅れていた shadow DOM も問題なく使える…！  
そこで足を引っ張るのはおなじみの IE11。2025 年まで存命予定のブラウザを現時点で切るのは流石に難しいので、これは Polyfill で対応するしかないです。

# WebComponents の Polyfill を使う

（主に）IE/Edge 向けに [WebComponents Polyfills](https://www.webcomponents.org/polyfills) が提供されています。  
これを導入すれば WebComponents がそのまま使えるようになるかと言うとそうでもなく、もう二手間ほど対応が必要になります。

## 手間その 1. ES5 へトランスパイルする

WebComponents は ES2016 のクラス構文が前提となっているので、そのままでは IE11 は動作しません。WebComponents のコードを ES5 にトランパイルする必要があります。

トランスパイルと言えば [Babel](https://babeljs.io) ですが、実はここに落とし穴があります。  
[WebComponents Polyfills](https://www.webcomponents.org/polyfills) は未対応ブラウザに対して 自動的に Polyfill をあててくれるのですが、この処理と Babel の core-js の処理との相性が悪く、IE11 でスタックオーバーフローが発生してしまいます。  
この不具合は Issue としてはあるものの、未だ解決には至らない様子。
https://github.com/webcomponents/polyfills/issues/43

このスタックオーバーフローは Symbol の Polyfill が原因のようで、別途 Symbol の Polyfill （例えば https://polyfill.io/v3/polyfill.min.js?features=Symbol など）を当てることでこのエラーは一応は回避できますが、polyfill.io、WebComponents Polyfills、Babel(core-js) と 3 つの Polyfill を重ね掛けする形になります。

そこで、今回の WebComponents を IE11 で動かす目的としては **TypeScript を使うことを強くオススメします。**
polymer-cli の build コマンドも tsc を使っていることもあり、TypeScript の ES5 トランスパイルではこの問題に遭遇することなく動作します。

## 手間その 2. `ShadyCSS` で CSS のスコープを指定する

通常の WebComponents では以下のような記述になります。

```js:標準的なWebComponentsの書き方
class MyButton extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
      <style>
        button {
          border-radius: 5px;
        }
      </style>
      <button><slot></slot></button>
    `;
    this.addEventListener("click", e => {
      alert("click!!!");
    });
  }
}
customElements.define("my-button", MyButton);
```

ですが、IE/Edge ではこの記述のままだと style の定義がグローバルに撒け出てきてしまいます。これを回避する方法として[公式の使い方](https://github.com/webcomponents/polyfills/tree/master/packages/shadycss#usage) にもあるように、`ShadyCSS` を使って CSS のスコープが shadowRoot にあることを明示してあげる必要があります。
修正後のコードは以下の通りです。

```js:ShadyCSSを使った書き方
const html = document.createElement("template");
html.innerHTML = `
  <style>
    button {
      border-radius: 5px;
    }
  </style>
  <button><slot></slot></button>
`;

window.ShadyCSS && ShadyCSS.prepareTemplate(html, "my-button");

class MyButton extends HTMLElement {
  constructor() {
    super();
    this.addEventListener("click", e => {
      alert("click!!!");
    });
  }
  connectedCallback() {
    window.ShadyCSS && ShadyCSS.styleElement(this);
    if (!this.shadowRoot) {
      this.attachShadow({ mode: "open" });
      this.shadowRoot.appendChild(html.content.cloneNode(true));
    }
  }
}
customElements.define("my-button", MyButton);
```

# バニラ WebComponents はつらいのでライブラリを使いたい

WebComponents で HTML 周りの処理を書くと DOM API ベースで書くことになり、今のご時世では結構なつらみがあります。  
React.js や Vue.js の Virtual DOM による差分更新のように、レンダリングパフォーマンス最適化の恩恵にも与りたい…  
ということで、IE11 にも対応した WebComponents ライブラリを使うことにしましょう。

WebComponents を生成するライブラリといえば [Polymer Project](https://www.polymer-project.org/)。その中に WebComponents を生成する軽量クラスライブラリの [LitElement](https://lit-element.polymer-project.org/) があります。  
[lit-html](https://lit-html.polymer-project.org/) で書けるので DOM API で書くよりパフォーマンスが良いですし、[先の ShadyCSS の適用](#手間その-2-shadycss-で-css-のスコープを指定する)などもライブラリ側がやってくれるので、こちらの手間が減ります。  
LitElement で書くと以下のような記述でスッキリ書けます。

```js:LitElementを使った書き方
import { LitElement, html, css } from "lit-element";

class MyButton extends LitElement {
  static get styles() {
    return css`
      button {
        border-radius: 5px;
      }
    `;
  }
  render() {
    return html`
      <button @click=${() => alert("click!!!")}><slot></slot></button>
    `;
  }
}

customElements.define("my-button", MyButton);
```

公式のブラウザ互換表示にはちゃんと IE11 のアイコンもいるので安心して使えます。  
…と思いきや、LitElement の公式手順に則ってもすぐにちゃんと動いてくれなくて、実働させるまでに色々と試行錯誤がありました。  
自分がいくつかハマったポイントをここに記して行きます。

## ハマりポイント 1. WebComponents を呼ぶ script に `type="module"` は付けない

[LitElement の使い方](https://lit-element.polymer-project.org/guide/use) では、

```html
<script type="module" src="./js/my-button.js"></script>
```

と書いてあって、最初はその通りに書いてみたものの、JS ファイルが読み込まれません。
当たり前といえば当たり前なんですが、 `type="module"` は ES Modules 向けの記述で、IE11 は解釈できずに処理をスキップします。

そもそも ES5 にトランスパイルしているファイルなので、以下のように通常の JavaScript と同様の形で呼び出してあげる必要があります。

```html
<script src="./js/my-button.js"></script>
```

## ハマりポイント 2. loader のオプションで LitElement, lit-html をトランスパイルの対象に指定する

npm に登録されている LitElement, lit-html は ES2017 で書かれており、ES5 での提供がありません。
そのため IE11 で動かすためにはこれらを import 時にトランスパイルする必要があります。

しかし、Webpack の loader は通例的に `exclude` オプションで `/node_modules/` はトランパイルの対象外としていることが多く、そのままだと バンドルファイルに ES2017 のコードが混じってしまいます。
loader のオプションで、 LitElement, lit-html は例外的にトランスパイルの対象として通知する必要があります。

```js:ts-loaderの場合
module: {
  rules: [
    {
      test: /\.(js|ts)$/,
      use: 'ts-loader',
      exclude: /node_modules\/(?!(lit-html|lit-element))\//,
    },
  ],
},
```

```js:babel-loaderの場合
module: {
  rules: [
    {
      test: /\.js$/,
      use: 'babel-loader',
      exclude: /node_modules\/(?!(lit-html|lit-element))\//,
    },
  ],
},
```

## ハマりポイント 3. (Babel を使う場合) @babel/preset-env で useBuiltIns: 'entry' を指定する

Babel の v7.4 以降から、 `@babel/polyfill` が deprecated となり、 core-js を使うことが推奨されるようになりました。

[Babel7.4 で非推奨になった babel/polyfill の代替手段と設定方法
](https://aloerina01.github.io/blog/2019-06-21-1)

いくつかの代替方法がありますが、`useBuiltIns: 'usage'` ではうまく動かなかったので、今回は `useBuiltIns: 'entry'` を使います。

以下は babel の設定例です。

```js:babel.config.js
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: "ie 11",
        modules: false,
        useBuiltIns: "entry",
        corejs: 3
      }
    ]
  ]
};
```

エントリーポイントの JS ファイルの先頭に以下を呼んでおきます。

```js
import "core-js/stable";
import "regenerator-runtime/runtime";
```

# サンプルコード＆動作デモ

Webpack ＆ LitElement でどうにか IE11 に対応できないか、試行錯誤の結果のコードを以下にまとめました。

https://github.com/okamoai/lit-element-es5

動作デモは以下で確認できます。

https://rano-raraku.net/lit-element-es5/

# まとめ：IE11 向けの WebComponents は LitElement & TypeScript で書け

最初はいつもの構成で Webpack & babel-loader でがんばってましたが、IE11 で WebComponents する場合、相当な理由がない限りは Babel を使わずに TypeScript で書いた方が謎の不具合に遭遇せず、圧倒的に楽できます。TypeScript で書きましょう。

動作デモに添付した Webpack Bundle Analyzer のレポートを見ても分かりますが、生成されたコードも圧倒的に TypeScript の方が軽いです。

- Babel: 146KB (Gzip: 42.92KB)
- TypeScript: 28.8KB (Gzip: 8.05KB)

これは Babel の オプションが `useBuiltIns: 'usage'` で、Polyfill 全乗っけ状態なので当然といえば当然ですが。

ビルドに `polymer-cli` を使った方が早いという話もありますが、Webpack の splitChunks や Tree Sharking はやっぱり有用ですし、画像ファイルも WebComponents にバンドルしたい、とかなると、やっぱり Webpack で WebComponents をビルドできるメリットは大きいですね。
