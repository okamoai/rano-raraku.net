---
Title: "Web API の CSRF 対策まとめ"
Date: "2019-07-16"
slug: "web-api-csrf"
categories:
  - "セキュリティ"
  - "csrf"
---

セキュリティ脆弱性診断などでたまに CSRF について指摘されることがあります。
今まではトークン発行して対応すれば良いんでしょ？ と思ってましたが、SPA のように非同期通信が前提の場合はどう対処するべきなんだろう、と疑問が出たりしたので、調べてみた結果をまとめてみました。

## CSRF とは

Cross Site Request Forgeries（クロスサイトリクエストフォージェリ）の略で、
サービス利用者の正規権限を利用して、意図しないタイミングでサービスの機能を実行させる攻撃手法のことを指します。
2005 年に mixi 日記で発生した「ぼくはまちちゃん」で一躍有名になりました。

[大量の「はまちちゃん」を生み出した CSRF の脆弱性とは？ - ITmedia エンタープライズ ](https://www.itmedia.co.jp/enterprise/articles/0504/23/news005.html)

## CSRF が発生する原因

サービスの機能を実行するプログラムへのリクエストの検証が権限情報のみであった場合に発生します。

![CSRF-1.png](/img/201907-CSRF-1.png)

CSRF は正規ユーザの権限を使って実行されるので権限情報のみの検証では不十分です。
権限情報の他にも正規のルートかつ正規のタイミングであるかを同時に検証する必要があります。

## 既存の API が CSRF 対策されているかチェックする

攻撃者が他サイトから正規ユーザのアクセスを利用して API に直接リクエストを送る方法は大きく分けて 2 つあります。

1. JavaScript（非同期通信）によるリクエスト
2. ブラウザの標準動作（同期通信）によるリクエスト

例えば、https://example.com/api/message/update という 更新 API に `text` というパラメータでリクエストを送るケースの場合、
認証済みの状態で以下の内容を含むページにアクセスをして、更新が実行されてしまうと CSRF 対策がなされてないことになります。

### 1. JavaScript（非同期通信）によるリクエスト

Web API の場合、ほとんどは JavaScript による非同期通信による送信が使われることになります。
ページに以下のような内容を仕込んでおくと API に向けてデータを送信することができます

```html
<script>
  var xhr = new XMLHttpRequest();
  xhr.open("POST", "https://example.com/api/message/update");
  xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
  xhr.withCredentials = true;
  xhr.send("text=csrf%20test");
</script>
```

JavaScript には [Same Origin Policy](https://developer.mozilla.org/ja/docs/Web/Security/Same-origin_policy) という仕様があり、クロスドメインで非同期通信を実行すると CORS policy エラーが発生します。
ドメインを跨いだ JavaScript の通信に制限がかかるため API 側がレスポンスに `Access-Control-Allow-Origin` ヘッダを付与してクロスドメインの通信を許可しない限り、JavaScript はレスポンスの内容を読み取ることができません。

一見この仕様により CSRF 対策になるのでは…？ と思われますが、Same Origin Policy はクロスドメインの JavaScript のレスポンス情報をブラウザが読み取らせないようにしているだけで、**リクエスト自体は API に到達しています。** CSRF はリクエストが成立すればレスポンスは不要なので、結局 API 側は更新を実行する前に認証情報の以外のチェックを行う必要があります。

### 2. ブラウザの標準動作（同期通信）によるリクエスト

例えば `<form>` 要素による送信です。実際に実行する際には不可視の iframe 内で処理するなどしてユーザに実行を気づかせないようにします。

```html
<form action="https://example.com/api/message/update" method="POST">
  <input type="hidden" name="text" value="csrf test" />
</form>
<script>
  document.forms[0].submit();
</script>
```

また API が GET を受け付けている場合は `<img>` 要素に差し込んでリクエストを送ることもできます。

```html
<img src="https://example.com/api/message/update?message=csrf%20test" />
```

また、同期通信は Same Origin Policy の制約を受けませんが、通信後の内容は正規サイトの管轄となるため、同期通信では外部サイトから正規サイトの情報にアクセスすることができません。

## CSRF の対策方法

CSRF 対策として大きく分けて 2 つのアプローチがあります。

1. トークンを発行してリクエストの正当性を検証してから実行する
2. プリフライトリクエストを検証してから実行する

### 1. トークンによる対策

事前にトークンを発行しておき、更新 API では認証情報のほかにトークンチェックを行い、正規のリクエストであるかを検証します。  
外部サイトからはトークン API へのリクエストは送れても、CORS エラーにより発行されたトークンを読み取ることができません。  
更新 API のリクエストも正規に発行されたトークンが必要になり、外部サイトからはトークンの内容を知る術がないため、リクエストは失敗します。

![CSRF-2.png](/img/201907-CSRF-2.png)

この仕組みは CSRF 対策としては古くから使われていて、既存のフレームワークの仕組みに取り込まれていたりしているため、分かり易い仕組みではあるものの、

- サーバ側にトークンを保存する必要がある
- クライアント側の API アクセス前にトークン API への手続き処理が必要になる

といった手間がありそれらの開発・運用をするためのコストが発生してしまうのがデメリットです。

### 2. プリフライトリクエスト検証による対策

ブラウザは JavaScript でクロスドメインの非同期通信を行う際に一定の条件を満たすと実際の送信の前に `OPTIOIN` メソッドを使って プリフライトリクエストを発行します。（一定の条件の詳細は [プリフライトリクエスト - MDN](https://developer.mozilla.org/ja/docs/Web/HTTP/CORS#Preflighted_requests) を参照してください）
この プリフライトリクエストが正規のルートからのリクエストであるかを検証することで CSRF 対策 とすることができます。

![CSRF-3.png](/img/201907-CSRF-3.png)

CSRF 対策としてプリフライトリクエストを発生させるには、カスタムヘッダを付与する方法が一般的です。また、リクエストの検証は以下の条件をチェックします。

1. 付与したカスタムヘッダが存在すること
2. `Host` ヘッダが正規サービスのホスト名であること
3. `Origin` ヘッダが許可した接続元のホスト名（API のレスポンスヘッダに含まれる `Access-Control-Allow-Origin`ヘッダと同じ）であること

これらの条件を満たさないリクエストはエラーとして返却することで、本来のリクエスト処理は中断され実行されなくなります。  
また、`<form>`による同期送信はクロスドメインにおいてもプリフライトリクエストを発生させずに本来のリクエストを直接送信することはできますが、同期送信はカスタムヘッダを付与できないため、リクエストチェックの条件「付与したカスタムヘッダが存在すること」を満たせずエラーとなります。  
結果として意図しない外部サイトからのリクエストを全て弾き、CSRF の成立を阻むことができます。

プリフライトリクエストによる対策がトークンによる対策よりも優れているのは、

- サーバ側でトークン API の開発やトークン管理が不要
- 既存の API に一律的に `OPTION` メソッドの受付と検証処理を追加するだけでよい
- クライアント側でトークン取得や送信の処理が不要
- クライアント側は API にカスタムヘッダを付与するだけでよい

といった点です。トークンによる対策よりも開発コスト、運用コストを削減できます。  
逆にデメリットとしては同期通信（`<form>`による送信）に対応できない点です。  
SPA 構成のサービスなどでは、想定外（JavaScript 以外から）のアクセスを一律排除できるので寧ろメリットになりますが。

## 動作検証サンプル

CSRF の対策が行われていない API と、プリフライトリクエストによる CSRF 対策が行われている API の動作サンプルを作ってみました。

https://github.com/okamoai/example-csrf

サンプルの詳細はリポジトリの README.md をご確認ください。

## まとめ

これから既存の API、もしくは新規のサービス開発で CSRF 対策を求められた場合、

- API との非同期通信がメインになる SPA のようなプロジェクトではプリフライトリクエストによる対策
- 同期的な送信で処理される従来型のプロジェクトではトークンによる対策

で対応するのが良さそうです。

トークンによる対策は SPA のプロジェクトにも適用はできますが、プリフライトリクエストで対応した方がフロントエンド、バックエンド共に対応コストが少なくて良さそうですね。

## Cookie の SameSite オプション【2019.09.12 追記】

Cookie には `SameSite` というオプションがあり、このオプションに `Lax` が指定されると、Cookie の `domain` オプションに指定されたドメイン以外からの Form POST/img/iframe/Ajax のリクエストについて Cookie を送信しなくなります。  
つまり外部サイトが CSRF 攻撃を行っても 対象の API には 認証情報の Cookie が送信されなくなるため、CSRF が成立しなくなります。

SameSite の対応状況  
https://caniuse.com/#search=samesite

Win10 以外の IE11 と Android4.4 の標準ブラウザを除きモダンブラウザは対応済みなため、実用性は十分。  
API のリクエストが Cookie の `domain` オプションの範囲に限定されるケースに限っては認証 Cookie に `SameSite=Lax` オプションを指定して CSRF 対策するのが最も低コストになります。

逆に API リクエストがクロスドメイン前提である場合は、前述のプリフライトリクエストによる CSRF 対策を行う必要があります。  
Chrome 80 からは `SameSite=Lax` が Cookie 生成時のデフォルトの値になるため、クロスドメインでの Cookie 送信が必要な場合は Cookie 生成時に明示的に `SameSite=None` を指定する必要があります。  
また、 `SameSite=None` 指定時には `secure` オプションも同時に有効にしないと適用されないため、注意が必要です。

## 参考

- [この Web API って CSRF 対策出来てますか？って質問にこたえよう](https://qiita.com/maruloop/items/e14d02299bd136f4b1fc)
- [独自ヘッダをチェックするだけのステートレスな CSRF 対策は有効なのか？](http://blog.a-way-out.net/blog/2015/03/23/stateless-csrf-protection/)
- [WebAPI のステートレスな CSRF 対策](https://momijiame.hatenadiary.org/entry/20111204/1323000833)
- [XMLHttpRequest を使った CSRF 対策](http://hasegawa.hatenablog.com/entry/20130302/p1)
- [Cookie の性質を利用した攻撃と Same Site Cookie の効果](https://blog.jxck.io/entries/2018-10-26/same-site-cookie.html)
- [Cookies default to SameSite=Lax](https://www.chromestatus.com/feature/5088147346030592)
- [Reject insecure SameSite=None cookies
  ](https://www.chromestatus.com/feature/5633521622188032)
