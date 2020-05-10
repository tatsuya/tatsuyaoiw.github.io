---
layout: post
title: Chrome Extensionの作り方
date: 2014-07-01 06:50
author: Tatsuya Oiwa
tags: [Japanese]
---

## 概要

ブラウザの右上にボタンを設置し、そのボタンをクリックするとページの背景を赤くするChromeのExtension（拡張機能）を作る。ソースは、[Sample Extensions]の[Page Redder]にある。

## 手順

1. `manifest.json`の作成
2. バックグラウンドスクリプトの作成
3. Extensionの読み込みと実行
4. CSSでページの背景を赤くする

### 1. manifest.jsonの作成

まず最初に、Extensionの設定ファイルである`manifest.json`を作成する。

```js
{
  // Required
  "manifest_version": 2,
  "name": "Page Redder",
  "version": "0.0.1",

  // Optional
  "browser_action": {
    "default_title": "Make this page red"
  },
  "background": {
    "scripts": [
      "background.js"
    ]
  },
  "permissions": [
    "activeTab"
  ]
}
```

上記のうち、必須となるパラメータは以下の3つ。

- `manifest_version` : `manifest.json`のフォーマットのバージョン。この記事を書いている時点でのバージョンは`2`。
- `name`: Extensionの名前。
- `version`: Extensionのバージョン。

その他のパラメータは、作りたいExtensionの種類によって異なるため、[Manifest File Format]のページを見ながら必要に応じて追加する。

なお、今回は**Browser Action**と呼ばれる種類のExtensionを作るので、次の3つのパラメータを追加した。

- `browser_action`: ツールバー上に表示するアイコンなどのUIの設定。
- `background`: アイコンがクリックされたときの動作を定義。
- `permissions`: Chromeの様々なAPIを利用するための権限の設定（詳細は[Declare Permissions]を参照）

### 2. バックグラウンドスクリプトの作成

設定ファイルの準備が整ったので、続いて`manifest.json`の`background.scripts`プロパティで指定したファイルである`background.js`を追加する。ファイルは`manifest.json`と同じディレクトリに置く。

ディレクトリ構成は以下の通り。

```
page-redder/
  manifest.json
  background.js
```

続いて、`background.js`を編集する。

```js
// Extensionのアイコンがクリックされたときに呼ばれる
chrome.browserAction.onClicked.addListener(function() {
  // デバッグコンソールにメッセージを表示
  console.log('hello world');
});
```

### 3. Extensionの読み込みと実行

次に、2で書いたプログラムを実際にChromeに読み込ませて実行してみる。

Extensionは最終的に`.crx`という形式にパッケージングした状態でウェブストアにリリースされるが、開発時に毎回その形式に変換→動作確認→コードを修正→変換...とするのは面倒である。そこで、ローカルのChromeにパッケージングしていない状態のプログラムを読み込んでテストを行う。

1. Chromeを起動し、URLバーから`chrome://extensions`を開く（または右上の設定メニューを開くボタンクリックし、Extensions > Toolsに移動する）。
2. 右上のDeveloper modeのチェックボックスがオンになっていることを確認する。
3. Load unpacked extension...をクリックし、`manifest.json`のあるディレクトリを選択して読み込む。

すると、ブラウザの右上に新しいアイコンが現れるので、そのアイコンにマウスオーバーしてみる。すると、先ほど`manifest.json`の`browser_action.default_title`プロパティで設定した**Make this page red**という名前が表示されれば、Extensionは正しく読み込まれている。

さらに`chrome://extensions`のページに戻りInspect views: background_pageというリンクをクリックすると、バックグラウンドで実行中のスクリプト用のデバッグコンソール別ウィンドウで表示される。そのコンソール上に**hello world**という文字列が表示されていれば、先ほどのコードが期待通りに動作していることがわかる。

### 4. CSSでページの背景を赤くする

3で動作確認ができたので、先ほどの`background.js`にコードを追加していく。

```js
// Extensionのアイコンがクリックされたときに呼ばれる
chrome.browserAction.onClicked.addListener(function(tab) {
  console.log('Turning ' + tab.url + ' red!');
  chrome.tabs.executeScript({
    code: 'document.body.style.backgroundColor="red"'
  });
});
```

もう一度`chrome://extensions`に戻りReloadボタンをクリックする。適当なページを開いた状態でExtensionのアイコンをクリックすると、ページの背景が赤くなれば成功となる。

なお、その他Chrome Extensionで利用可能なAPIについては[Developer's Guide]へ。

## 参考URL

- [Getting Started: Building a Chrome Extension] - 最も基本的なbrowser actionのチュートリアル
- [Overview] - browser action, page action, background, content scriptsなどのアーキテクチャの概要とChromeが提供するAPIについて
- [Developer's Guide] - APIの詳細ページ
- [Sample Extensions] - サンプル一覧


[Getting Started: Building a Chrome Extension]: https://developer.chrome.com/extensions/getstarted
[Overview]: https://developer.chrome.com/extensions/overview
[Sample Extensions]: https://developer.chrome.com/extensions/samples
[Page Redder]: https://developer.chrome.com/extensions/samples#page-redder
[Manifest File Format]: https://developer.chrome.com/extensions/manifest
[Declare Permissions]: https://developer.chrome.com/extensions/declare_permissions
[Developer's Guide]: https://developer.chrome.com/extensions/devguide
