---
layout: post
title: JavaScriptのプロトタイプチェーンについて
summary: JavaScriptのプロトタイプチェーン（Prototype Chain）やプロトタイプ継承（Prototype Inheritance）について熱く語ってます
date: 2015-01-21 08:11
---

## プロトタイプチェーンとは

すべてのJavaScriptオブジェクトは、そのオブジェクト自身が持つプロパティ（own properties）の他に、内部でプロトタイプ `[[Prototype]]` と呼ばれる別のオブジェクトへの参照を持つ。

例えば、あるオブジェクトに対し、`obj.name` のようにプロパティの値を取得しようとする場合を考える。もし、オブジェクトに `name` というプロパティが存在すればその値を返し、なければ、続けてオブジェクトの `[[Prototype]]` が参照しているオブジェクトのプロパティに `name` があるかどうかを探し始める。もし、`[[Prototype]]` が参照するオブジェクトにも `name` プロパティがない場合、さらにその次の `[[Prototype]]` へと探索は続き、最終的にプロパティが見つかる、もしくは `[[Prototype]]` の値が `null` になるまでこれを繰り返していく。

このように、プロパティの参照が別のオブジェクトへ次々とつながっていく仕組みのことを、プロトタイプチェーン（Prototype Chain）やプロトタイプ継承（Prototype Inheritance）と呼ぶ。

## `__proto__` について

いくつかのJavaScriptの実装では `__proto__` というプロパティを使って `[[Prototype]]` プロパティにアクセスすることができる。

例えば、次のコードでは、`__proto__` アクセサを使って、オブジェクトのプロトタイプを設定したり書き換えている。

```js
var person = {
  kind: 'person'
};

var alien = {
  kind: 'alien'
};

var bob = {};

bob.__proto__ = person;
console.log(bob.kind); //=> person

bob.__proto__ = alien;
console.log(bob.kind); //=> alien
```

## `Object.create()` について

`__proto__` はJavaScriptの実行環境によってはサポートされていないため、ES5以降では代わりに `Object.create()` を使って同様のことを実現できる。

```js
var person = {
  kind: 'person'
};

var bob = Object.create(person);

console.log(bob.kind); //=> person
```

## コンストラクタ関数を使ったプロトタイプの作成

先の `__proto__` や `Object.create()` を使ったプロトタイプの作成はあまり一般的ではない。代わりに最も頻繁に使われるのが、コンストラクタ関数を使った方法である。

例えば、`Person()` というコンストラクタ関数に対し `new` キーワードを使ってオブジェクトを生成する場合、生成されたオブジェクトの `[[Prototype]]` プロパティが参照するオブジェクトは `Person.prototype` となる。

```js
function Person(name) {
  this.name = name;
}

Person.prototype.kind = 'person';

var bob = new Person('Bob');

bob.__proto__ === Person.prototype //=> true

bob.kind //=> person
```

このように、コンストラクタ関数とプロトタイプを組み合わせることで、JavaScriptでも他のクラスベースの言語のような機能を実装することができる。一般的に、あるクラスの実装において、インスタンス間で共通して使われるようなメソッドはプロトタイプに定義しておき、インスタンスごとに異なるフィールドの値は各オブジェクトごとに持つようにすると良い。

以下は、`Person` クラスの場合の実装例である。

```js
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  console.log('Hi, I\'m ' + this.name);
};

var bob = new Person('Bob');
var mike = new Person('Mike');

bob.greet(); //=> Hi, I'm Bob
mike.greet(); //=> Hi, I'm Mike
```

上記の場合、`greet()` という共通のメソッドはプロトタイプで定義しているのに対し、インスタンスごとに異なる `name` の値は、各オブジェクトが別々に持つように実装されている。

## 参考

- [A Plain English Guide to JavaScript Prototypes - Sebastian's Blog](http://sporto.github.io/blog/2013/02/22/a-plain-english-guide-to-javascript-prototypes/)
- [dynamic languages - How does JavaScript .prototype work? - Stack Overflow](http://stackoverflow.com/questions/572897/how-does-javascript-prototype-work)
