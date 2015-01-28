---
layout: post
title: JavaScriptのargumentsオブジェクトについて
summary: JavaScriptの関数の引数リストを取得する arguments オブジェクトの説明と使用例
date: 2015-01-21 08:11
---

## argumentsオブジェクト

JavaScriptの関数では通常、仮引数を使って関数の呼び出し時に渡された引数の値を取得する。例えば、 `function f(x)` という関数を宣言した場合、`f(1)` という関数呼び出しによって渡された値 `1` は、仮引数 `x` によって参照が可能となる。

```js
function f(x) {
  console.log(x); //=> 1
}
f(1);
```

では、次のような場合はどうだろうか。

```js
function f(x) {
  console.log(x);
}
f(1, 2);
```

実は、JavaScriptでは、関数呼び出し時に渡された引数のリストを `arguments` というオブジェクトによって関数スコープ内から全て取得することができる。つまり、上記の例の場合、1つめの引数 `1` は `x` または `arguments[0]` によって参照することができ、2つめの引数 `2` は `arguments[1]` から参照することができる。

```js
function f(x) {
  console.log(x); //=> 1
  console.log(arguments[0]); //=> 1
  console.log(arguments[1]); //=> 2
}
f(1, 2);
```

## 関数に渡された引数の数を確認する

`arguments` オブジェクトは `length` プロパティを持っているので、関数内で実際に渡された引数の数を確認することができる。

```js
function f(x, y, z) {
  if (arguments.length != 3) {
    throw new Error("function f called with " + arguments.length + "arguments, but it expects 3 arguments.");
  }
}
```

## 可変長の引数をサポートする

`arguments` オブジェクトの別の使用例として、可変長の引数をサポートする方法がある。以下の関数 `max()` は、関数呼び出しによって渡されたすべての引数の中から最大の値を返す。

```js
function max(/* ... */) {
  var max = Number.NEGATIVE_INFINITY;
  for(var i = 0; i < arguments.length; i++) {
    if (arguments[i] > max) {
      max = arguments[i];
    }
  }
  return max;
}
var largest = max(1, 10, 100, 2, 3, 1000, 4, 5, 10000, 6); // => 10000
```

## Array-Likeなオブジェクト

`arguments` オブジェクトは `length` プロパティを持ち、あたかも配列のように振る舞うが、厳密には配列ではない。例えば、`arguments` オブジェクトは `Array.prototype` を継承しておらず、`Array.prototype.forEach` や `Array.prototype.map` 等のメソッドを使用することができない。このようなオブジェクトは **Array-Like** なオブジェクトと呼ばれ、`arguments` の他にも、例えばクライアントサイドのDOM操作のメソッド `document.getElementsByTagName()` の戻り値は Array-Like なオブジェクトである。

## Array-Likeなオブジェクトを配列のように扱う

`Array.prototype` のメソッド群は Array-Like なオブジェクトに対しても動作するよう汎用的に実装されているため、`Function.call` を使って通常の配列と同じように関数を適用することができる。

```js
function f() {
  Array.prototype.join.call(arguments, ', ') //=> 'a, b, c'
  Array.prototype.map.call(arguments, function(x) {
    return x.toUpperCase();
  }); //=> ["A", "B", "C"]
}
f('a', 'b', 'c');
```

また、`Array.prototype.slice` を使うと、`arguments` オブジェクトから配列のコピーを生成することができる。

```js
function f() {
  Array.prototype.slice.call(arguments, 0) //=> ["a", "b", "c"]
}
f('a', 'b', 'c');
```

