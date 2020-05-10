---
layout: post
title: Node.jsでモジュールを作ってみる
date: 2013-05-08 18:19:00
author: Tatsuya Oiwa
tags: [Japanese]
---

## はじめに

`require` とか `exports` とか `module.exports` とか最初よくわからなかったのでまとめた。

## モジュールを作る

日本円 <=> 米ドル間の通貨計算をするcurrencyモジュールを作ってみる。currencyモジュールには二つの関数 `JPYToUSD(yen)` と `USDToJPY(usd)` を作成し、これらの関数を別のスクリプトから呼び出せるようにする。

```js
function JPYToUSD(yen){
  // return usd
}

function USDToJPY(usd){
  // return yen
}
```

Node.js でモジュールを作る方法は大きく二つある。

- 単体のファイルとして作る方法
- ディレクトリとして作る方法（ディレクトリ内の複数のファイルを一つのモジュールとして扱う方法）

小さなモジュールであれば単体のファイルで十分だが、モジュールの機能が増えてくるとファイルを分割したほうがメンテナンスしやすくなる場合もある。

なお、ディレクトリで作成する際は、ディレクトリの直下に `index.js` という名前のファイルを作ることで、Nodeが自動的にそのファイルを読み込む。

```
currency.js // ファイル
currency/index.js // ディレクトリ
```

では、先ほどの二つの関数をモジュールとして呼び出せるようにする。

モジュールの作成は `exports` という名前のオブジェクトに、string や object, function などの値をプロパティとして持たせることで、それらを外部から参照できるようにすることができる。今回の場合、関数 `JPYToUSD()` と関数 `USDToJPY()` を外部から呼び出せるようにしたいので、`exports.JPYToUSD`, `exports.USDToJPY` という風に書けばOK。

```js
// private
var JPY = 0.0098;

// private
function roundTwoDecimals(amount) {
  return Math.round(amount * 100) / 100;
}

// public
exports.JPYToUSD = function(yen) {
  return roundTwoDecimals(yen * JPY);
}

// public
exports.USDToJPY = function(us) {
  return roundTwoDecimals(us / JPY);
}
```

上記の場合、`exports` オブジェクトのプロパティである二つの関数、`JPYToUSD` と `USDToJPY` はPublic（外部から参照可能）なのに対し、`roundTwoDecimals` や `JPY`（変換レート）は、モジュール内部でしか使われないためPrivate（外部から参照不可）となっている。

## モジュールを読み込む

作成したモジュールを読み込むには、Nodeの `require` 関数を使う。

```js
var currency = require('./currency');

console.log(currency.JPYToUSD(100));
console.log(currency.USDToJPY(1));
```

`requre` 関数の引数にモジュール名（currency）を渡すと、モジュールが読み込まれる。モジュールファイル名の ".js" は省略が可能。また、モジュール名の最初が "./" から始まっているのは、そのモジュールが呼び出し元のスクリプトと同じディレクトリに存在していることを意味している。仮に `currency.js` を `lib` ディレクトリ配下に置くとすると、

    var currency = requre('./lib/currency');

と書き換えることで、同じようにモジュールを参照できる。

## module.exports を使う

これまで、`JPYToUSD` と `USDToJPY` という二つの関数を外部のスクリプトから直接参照できるようにしていたが、JavaScript の Prototype を使うことで以下のように書き換えることもできる。

```js
var Currency = function(JPY) {
  this.JPY = JPY;
}

var roundTwoDecimals = function(amount){
  return Math.round(amount * 100) / 100;
}

Currency.prototype.JPYToUSD = function(yen){
  return roundTwoDecimals(yen * this.JPY);
}

Currency.prototype.USDToJPY = function(us){
  return roundTwoDecimals(us / this.JPY);
}

exports = Currency;
```

モジュールを読み込む側の例。

```js
var Currency = require('./currency')
  , JPY = 0.0098;

currency = new Currency(JPY);
console.log(currency.JPYToUSD(100));
```

新しい `currency.js` では、関数 `Currency` の `prototype` プロパティに関数を定義することで、`Currency` をクラス（およびコンストラクタ）のように定義できるようになった。

この場合、外部からは `Currency` クラスのみ参照が可能となるようにしたいので、`exports = Currency` としているが、実際はうまくいかない。なぜなら、`exports` プロパティには Object 型以外の値を入れることが禁止されており、TypeError例外が発生しまうためです。

そこで、この問題を回避するために `module.exports` を使う。

    module.exports = Currency;

このように `exports` オブジェクトを直接上書きすることで、`Currency` クラスを直接渡せるようになった。

ただし、モジュール内に `exports` と `module.exports` の両方を定義した場合は、`module.exports` が優先されて、exportsは無視されてしまうので、注意が必要となる。
