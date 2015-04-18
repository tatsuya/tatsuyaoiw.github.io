---
layout: post
title: Node.jsのconsole.logとprocess.stdout.writeの違い
summary: 果たして違いはあるのか
date: 2015-02-02 08:52
---

`console.log` は、関数呼び出し時に渡された引数の値をコンソール上に標準出力するメソッドである。

```js
console.log('hello world'); //=> hello world
```

これとほぼ同じ用途で使われるメソッドに `process.stdout.write` がある。

```js
process.stdout.write('hello world'); //=> hello world
```

両者の違いは何なのかというと、実は `console.log` はその関数内で `process.stdout.write` を呼んでいる。以下は、[joyent/node] の実際のコードである。

```js
Console.prototype.log = function() {
  this._stdout.write(util.format.apply(this, arguments) + '\n');
};
```

ただし、両者には大きく2つの違いがあることがわかる。

1つめは、`console.log` は出力の最後に改行 `\n` を追加する。そのため、意図して改行を外したい場合には、代わりに `process.stdout.write` を使うと良い。

2つめは、`console.log` は [`util.format`][util.format] を使って文字列をフォーマットしている。そのため、`console.log` では、次のようにフォーマット用の文字列とフォーマットした値を任意の数だけ渡して使うことができる。

```js
console.log('count: %d', count);
```

参考: [javascript - Difference between "process.stdout.write" and "console.log" in node.js? - Stack Overflow][stackoverflow]

[joyent/node]: https://github.com/joyent/node/blob/master/lib%2Fconsole.js#L55
[stackoverflow]: http://stackoverflow.com/questions/4976466/difference-between-process-stdout-write-and-console-log-in-node-js
[util.format]: http://nodejs.org/api/util.html#util_util_format_format
