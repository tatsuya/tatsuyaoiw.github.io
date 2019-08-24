---
layout: post
title: JavaScriptの巻き上げ（ホイスティング）について
summary: JavaScriptの巻き上げ（ホイスティング - Hoisting）についてのメモ
date: 2015-01-19 18:19
categories: [Japanese]
---

Javascriptで変数を宣言するとき、その変数宣言はスコープの一番上へと巻き上げ（ホイスティング - Hoisting）られる。

例えば、以下のようなコードの場合、

```js
function example() {
  console.log(notDefined); // ReferenceError: notDefined is not defined
}
```

変数`notDefined`は宣言されていないため(グローバルに存在する場合を除いて）`ReferenceError`となる。一方、次のようなコードの場合、

```js
function example() {
  console.log(declaredButNotAssigned); // undefined
  var declaredButNotAssigned = true;
}
```

変数`declaredButNotAssigned`は、その変数への参照（`console.log(declaredButNotAssigned)`）より後に宣言されているにも関わらず、このコードは`ReferenceError`とはならない。これは、JavaScriptのインタプリタが変数宣言をその変数のスコープの最上部に巻き上げて（Hoist）いるためである。しかし、変数への代入は巻き上げの対象ではないため、`declaredButNotAssigned`の値は`true`ではなく`undefined`となる。

この特徴を考慮すると、上記のコードは以下のように書き変えることもできる。

```js
function example() {
  var declaredButNotAssigned;
  console.log(declaredButNotAssigned);
  declaredButNotAssigned = true;
}
```

変数の値が無名関数の場合、変数宣言は巻き上げられ、関数の代入は巻き上げられない。

```js
function example() {
  console.log(anonymous); // undefined

  anonymous(); // TypeError anonymous is not a function

  var anonymous = function() {
    console.log('anonymous function expression')
  }
}
```

変数の値が名前付き関数の場合も同様、変数宣言のみが巻き上げられ、関数名や関数の本体は巻き上げられない。

```js
function example() {
  console.log(named); // undefined

  named(); // TypeError named is not a function

  superPower(); // ReferenceError superPower is not defined

  var named = function superPower() {
    console.log('Flying');
  }
}
```

変数名と関数名が同じ場合も同様、変数宣言のみが巻き上げられる。

```js
function example() {
  console.log(named); // undefined

  named(); // TypeError named is not a function

  var named = function named() {
    console.log('named');
  }
}
```

ただし、関数宣言は巻き上げの対象となる。

```js
function example() {
  superPower(); // Flying

  function superPower() {
    console.log('Flying');
  }
}
```

参考: [airbnb/javascript]

[airbnb/javascript]: https://github.com/airbnb/javascript#hoisting
