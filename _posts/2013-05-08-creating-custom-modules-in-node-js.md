---
layout: post
title:  "Node.jsでモジュールを作ってみる"
date:   2013-05-08 18:19:00
---

`require` とか `exports` とか `module.exports` とか最初よくわからなかったのでまとめました。

## モジュールを作る

日本円 <=> 米ドル間の通貨計算をする currency モジュールを作ってみます。currency モジュールには二つの関数 `function JPYToUSD(yen)` と `function USDToJPY(usd)` を作成し、これらの関数を別のスクリプトから呼び出せるようしてみたいと思います。

    function JPYToUSD(yen){
      // return usd
    }
    
    function USDToJPY(usd){
      // return yen
    }

Node.js でモジュールを作る方法は大きく二つあります。一つは、単体のファイルとして作る方法、もう一つはディレクトリとして作る方法（ディレクトリ内の複数のファイルを一つのモジュールとして扱う方法）です。小さなモジュールであれば単体のファイルで十分ですが、モジュールの機能が増えてくるとファイルを分割したほうがメンテナンスしやすくなる場合もあるので、そのような場合はディレクトリのほうが便利です。

なお、ディレクトリで作成する際は、ディレクトリの直下に `index.js` という名前のファイルを作ることで、Node が自動的にそのファイルを読み込んでくれるようになります。

    currency.js // ファイル
    currency/index.js // ディレクトリ

では、先ほどの二つの関数をモジュールとして呼び出せるようにします。モジュールの作成は `exports` という名前のオブジェクトに、string や object, function などの値をプロパティとして持たせることで、それらを外部から参照できるようにすることができます。今回の場合、関数 `JPYToUSD` と関数 `USDToJPY` を外部から呼び出せるようにするので、`exports.JPYToUSD`, `exports.USDToJPY` という風に書けばOKです。

    var JPY = 0.0098;  // private
    
    function roundTwoDecimals(amount){  // private
      return Math.round(amount * 100) / 100;
    }
    
    exports.JPYToUSD = function(yen){  // public
      return roundTwoDecimals(yen * JPY);
    }
    
    exports.USDToJPY = function(us){   // public
      return roundTwoDecimals(us / JPY);
    }

上記の場合、`exports` のプロパティである二つの関数、`JPYToUSD` と `USDToJPY` はPublic（外部から参照が可能）なのに対し、`roundTwoDecimals` や `JPY`（変換レート）は、モジュール内部でしか使わないのでPrivateのままにしてあります。

## モジュールを読み込む

作成したモジュールを読み込むには、Nodeの `require` 関数を使います。

    var currency = require('./currency');

    console.log(currency.JPYToUSD(100));
    console.log(currency.USDToJPY(1));

`requre` 関数の引数にモジュール名（currency）を渡すと、モジュールが読み込まれます。モジュールファイル名の ".js" は省略可能です。また、モジュール名の最初が "./" から始まっているのは、そのモジュールが呼び出し元のスクリプトと同じディレクトリに存在していることを意味しています。仮に `currency.js` を `lib` ディレクトリ配下に置くとすると、

    var currency = requre('./lib/currency');

と書き換えることで、同じようにモジュールを参照できます。

## module.exports を使う

先ほど例では、`JPYToUSD` と `USDToJPY` という二つの関数を外部のスクリプトから直接参照できるようにしていましたが、JavaScript の Prototype を使うことで以下のように書き換えることもできます。

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

呼び出し側のスクリプトの例。

    var Currency = require('./currency')
      , JPY = 0.0098;

    currency = new Currency(JPY);
    console.log(currency.JPYToUSD(100));

新しい `currency.js` では、関数 `Currency` の `prototype` プロパティに関数を定義することで、`Currency` をクラス（およびコンストラクタ）のように定義できるようになりました。

この場合、外部からは `Currency` クラスだけが呼ばれるようにしたいので、`exports = Currency` としていますが、実際はうまくいきません。なぜなら、`exports` プロパティには Object 型以外を値を入れられないため、TypeError例外が発生しまうためです。

そこで、この問題を回避するために `module.exports` を使います。

    module.exports = Currency;

このように `exports` オブジェクトを直接上書きすることで、`Currency` クラスを直接渡せるようになりました。

ただし、モジュール内に `exports` と `module.exports` の両方を定義した場合は、`module.exports` が優先されて、exportsは無視されてしまうので、注意が必要です。
