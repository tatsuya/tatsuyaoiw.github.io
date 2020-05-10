---
layout: post
title: Node.jsでオブジェクト（モジュール）をextendする
summary: underscore.extend や util._extend を使って既存モジュールを拡張する方法など
date: 2015-02-09 18:24
tags: [Japanese]
---

調べた結果、[underscore] の [extend] か [joyent/node] の [util._extend] を使うのが良さそう。

## underscore.extend

```js
// Extend a given object with all the properties in passed-in object(s)
_.extend = function(obj) {
  if (!_.isObject(obj)) return obj;
  var source, prop;
  for (var i = 1, length = arguments.length; i < length; i++) {
    source = arguments[i];
    for (prop in source) {
      if (hasOwnProperty.call(source, prop)) {
        obj[prop] = source[prop];
      }
    }
  }
  return obj;
};

// Is a given variable an object?
_.isObject = function(obj) {
  var type = typeof obj;
  return type === 'function' || type === 'object' && !!obj;
};
```

- 良い点: わかりやすい
- 悪い点: 依存モジュールが増える

## util._extend

```js
exports._extend = function(origin, add) {
  // Don't do anything if add isn't an object
  if (!add || !isObject(add)) return origin;

  var keys = Object.keys(add);
  var i = keys.length;
  while (i--) {
    origin[keys[i]] = add[keys[i]];
  }
  return origin;
};

function isObject(arg) {
  return typeof arg === 'object' && arg !== null;
}
```

- 良い点: 依存なし
- 悪い点: `util._extend` はpublicではないので、将来消える可能性も無くはない

## 実装例

例えば、Nodeのビルトインモジュールである [util] に、新たに `isNumeric` という関数を追加する場合、以下のような実装になる。

### underscore.extend

```js
var _ = require('underscore');
var util = require('util');

var myUtil = _.extend({}, util);

myUtil.isNumeric = function(val) {
  return typeof val !== 'object' && val - parseFloat(val) >= 0;
};

module.exports = myUtil;
```

### util._extend

```js
var util = require('util');

var myUtil = util._extend({}, util);

myUtil.isNumeric = function(val) {
  return typeof val !== 'object' && val - parseFloat(val) >= 0;
};

module.exports = myUtil;
```

あとは、underscore と util を丸ごとくっつけて、一つの util モジュールにする方法もある。

```js
var _ = require('underscore');
var util = require('util');

var myUtil = _.extend({}, _, util);

module.exports = myUtil;
```

コード: [tatsuyaoiw/node-extend-example]

[underscore]: http://underscorejs.org
[extend]: http://underscorejs.org/#extend
[joyent/node]: https://github.com/joyent/node/
[util]: http://nodejs.org/api/util.html
[util._extend]: https://github.com/joyent/node/blob/master/lib%2Futil.js
[tatsuyaoiw/node-extend-example]: https://github.com/tatsuyaoiw/node-extend-example