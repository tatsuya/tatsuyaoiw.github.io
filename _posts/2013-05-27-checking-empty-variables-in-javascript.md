---
layout: post
title: JavaScript (Node.js) で空のオブジェクトを評価する
date: 2013-05-27 18:27:00
tags: [Japanese]
---

JavaScriptで条件分岐を書くときのために。

## if による判定　(undefined, null, false, 0, '')

if文による判定の場合、`undefined`, `null`, `false`, `0`, `''`(空文字) は`false`になる。ちなみに`-1`は`true`なので注意。

```js
console.log( Object ? true : false);    // true
console.log( undefined ? true : false); // false
console.log( null ? true : false);      // false
console.log( true ? true : false);      // true
console.log( false ? true : false);     // false
console.log( 0 ? true : false);         // false
console.log( 1 ? true : false);         // true
console.log( -1 ? true : false);        // true
console.log( '' ? true : false);        // false
console.log( 'a' ? true : false);       // true
console.log( [] ? true : false);        // true
console.log( {} ? true : false);        // true
```

## 配列が空かどうかを判定

if文だと配列が空かどうかは判定できないので、lengthをとる。（`[].length`は`0`）

```js
console.log( [].length ? true : false);    // false
console.log( ['a'].length ? true : false); // true
```

## オブジェクト（ハッシュ）が空かどうか判定

オブジェクトにはlengthメソッドが使えないので、Object.keys() で全てのキーから配列を生成した後、配列と同様 length で判定する。

```js
console.log( Object.keys({}).length ? true : false);
console.log( Object.keys({a:'a'}).length ? true : false);
```
