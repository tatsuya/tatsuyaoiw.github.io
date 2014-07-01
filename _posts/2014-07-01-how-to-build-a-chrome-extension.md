---
layout: post
title: "Chrome Extensionの作り方"
date:  2014-07-01 06:50
---

## はじめに

Chrome Extensionの[Sample Extensions](https://developer.chrome.com/extensions/samples)のページで紹介されている[Page Redder](https://developer.chrome.com/extensions/samples#page-redder)（ページの背景を赤くする）を参考にしながら作ってみた。


## manifest.jsonの作成

まずは、Chrome Extensionに必須の設定ファイルである`manifest.json`を作成する。

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

`manifest_version`, `name`, `version`の3つのプロパティは必須となる。

- `manifest_version` - `manifest.json`のフォーマットのバージョン
- `name` - Extensionの名前
- `version` - Extensionのバージョン

その他に、ツールバー上に表示するアイコンなどのUIを設定するためのプロパティである`browser_action`、アイコンがクリックされたときの動作を定義するスクリプトのファイルを指定した`background`、さらにChromeの様々なAPIを利用する際に指定する`permissions`プロパティ（詳細は[Declare Permissions](https://developer.chrome.com/extensions/declare_permissions)を参照）を設定した。

`manifest.json`で定義可能なその他のプロパティについては[Manifest File Format](https://developer.chrome.com/extensions/manifest)を参照。

## コーディング

設定ファイルの準備が整ったので、続いて`manifest.json`の`background.scripts`で指定した`background.js`を作成しコードを書いていく。

```js
// Extensionのアイコンがクリックされたときに呼ばれる
chrome.browserAction.onClicked.addListener(function() {
  // デバッグコンソールにメッセージを表示
  console.log('hello world');
});
```

## Extensionの読み込みと実行

Extensionは最終的に`.crx`という形式にパッケージングした状態でChromeのウェブストアにリリースされるが、開発時に毎回その形式に変換するのは面倒なので、そのままの状態でChromeに読み込んでテストを行う。

- Chromeを起動し、URLバーからchrome://extensionsを開く（または右上の設定メニューを開くボタンクリックし、Extensions > Toolsに移動する）。
- 右上のDeveloper modeのチェックボックスがオンになっていることを確認する。
- Load unpacked extension...をクリックし、manifest.jsonのあるディレクトリを選択して読み込む。

ここまでの手順を行っていくと、ブラウザの右上に新しいアイコンが現れる。そのアイコンにマウスオーバーすると、先ほど`manifest.json`の`browser_action.default_title`で定義した"Make this page red"という文字列が表示される。

さらにchrome://extensionsのページに戻りInspect views: background_pageというリンクをクリックすると、backgroundで実行中のスクリプト用のデバッグコンソールがポップアップで表示される。そのコンソール上にhello worldが表示されていれば、Extensionは期待通りに動作している。

## コーディングの続き

ここまでログを出力するのみだった部分を、実際のコードに置き換えていく（Chrome Extensionで利用可能なAPIについては[Developer's Guide](https://developer.chrome.com/extensions/devguide)を参照）。

```js
// Extensionのアイコンがクリックされたときに呼ばれる
chrome.browserAction.onClicked.addListener(function(tab) {
  console.log('Turning ' + tab.url + ' red!');
  chrome.tabs.executeScript({
    code: 'document.body.style.backgroundColor="red"'
  });
});
```

もう一度chrome://extensionsに戻ってReloadし、適当なページを開いた状態でExtensionのアイコンをクリックすると、ページの背景が赤くなる。

## リンク

- [Getting Started: Building a Chrome Extension](https://developer.chrome.com/extensions/getstarted) - 最も基本的なbrowser actionのチュートリアル
- [Overview](https://developer.chrome.com/extensions/overview) - browser action, page action, background, content scriptsなどのアーキテクチャの概要とChromeが提供するAPIについて
- [Developer's Guide](https://developer.chrome.com/extensions/devguide) - APIの詳細ページ
- [Sample Extensions](https://developer.chrome.com/extensions/samples) - サンプル一覧





