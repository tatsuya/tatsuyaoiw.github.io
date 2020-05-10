---
layout: post
title: Node.jsのEventsとEventEmitter
date: 2014-07-23 19:49
author: Tatsuya Oiwa
tags: [Japanese]
---

Node.jsの[Events](http://nodejs.org/api/events.html)モジュールを使って、独自のイベントを作成することができる。

例えば、あるボタンが押されたときにLEDのランプを点滅させたいとき、以下のように実装する。

```js
var events = require('events');

var eventEmitter = new events.EventEmitter();

var blink = function() {
  console.log('blinking');
}

eventEmitter.on('pressButton', blink);

eventEmitter.emit('pressButton');
```

まず、`require('events')`でEventsモジュールを読み込み、`EventEmitter`クラスのインスタンスを作成する。次に、`blink()`という関数内で、LEDランプを点滅させるコードを書き（実際には'blinking'という文字列を表示するだけ）、イベントが発生したらこの関数が呼ばれるようにする。

そして、先ほど作成した`EventEmitter`のインスタンスの`on()`関数で、第一引数にイベント名となる文字列`pressButton`、第二引数に、そのイベントが発生した時に呼ばれる関数`blink()`を渡す。

最後に、`eventEmitter.emit()`関数を実行すると、先ほどリスナーとして登録した関数が呼ばれる。

また、一つのイベントに対して複数のリスナーを登録することもできる。

```js
var events = require('events');

var eventEmitter = new events.EventEmitter();

var blinkOne = function() {
  console.log('blinking one');
}
var blinkTwo = function() {
  console.log('blinking two');
}
var blinkThree = function() {
  console.log('blinking three');
}

eventEmitter.on('pressButton', blinkOne);
eventEmitter.on('pressButton', blinkTwo);
eventEmitter.on('pressButton', blinkThree);

eventEmitter.emit('pressButton');
```

さらに、リスナーの関数に対して引数を渡したい場合は、`emit()`関数の第二引数以降に値を渡すことができる。

```js
var events = require('events');

var eventEmitter = new events.EventEmitter();

var blink = function(led) {
  console.log('blinking ' + led);
}

eventEmitter.on('pressButton', blink);

eventEmitter.emit('pressButton', 'one');
```

なお、EventEmitterを使った別の手法として、EventEmitterクラスを継承する方法がある。

LEDランプのプログラムをこの方法に書き換えるために、新たに`Button`という名前のクラスを作成し、`press()`という関数から`pressButton`イベントを呼べるようにする。

```js
var events = require('events');
var util = require('util');

function Button(led) {
  this.led = led;
  events.EventEmitter.call(this);
}

util.inherits(Button, events.EventEmitter);

Button.prototype.press = function() {
  this.emit('press', this.led);
}

var blink = function(led) {
  console.log('blinking ' + led);
}

var buttonOne = new Button('one');
buttonOne.on('press', blink);
buttonOne.press();
```

`Button`オブジェクトのコンストラクタは、LEDランプの名前`led`を引数にとり、さらに`call()`関数で`EventEmitter`オブジェクトのコンストラクタを実行する。そして、Node.jsの`util`モジュールの`inherits()`関数を使って、`EventEmitter`のプロパティを`Button`オブジェクトにコピーする。すると、`Button`クラスのインスタンスから`EventEmitter`の`on()`や`emit()`などの関数を実行することができるようになる。


最後に、登録したリスナーは`removeListener()`で削除することができる。

```js
buttonOne.removeListener('press', blink);
```

その他のAPIについては、[公式ドキュメント](http://nodejs.org/api/events.html)を参照。


