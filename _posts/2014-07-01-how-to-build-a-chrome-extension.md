---
layout: post
title: "Chrome Extensionの作り方"
date:  2014-07-01 06:50
---

## はじめに

Chrome Extensionの[Sample Extensions](https://developer.chrome.com/extensions/samples)で紹介されている[Page Redder](https://developer.chrome.com/extensions/samples#page-redder)（ページの背景を赤くする拡張機能）を作る。


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

必須のパラメータは以下の3つ。

- `manifest_version` : `manifest.json`のフォーマットのバージョン。
- `name`: Extensionの名前
- `version`: Extensionのバージョン

その他必要なパラメータは、作りたいExtensionの種類によって変わるので、詳細は[Manifest File Format](https://developer.chrome.com/extensions/manifest)を参照してほしい。

今回はBrowser Actionと呼ばれるタイプのExtensionを作成するので、以下の3つのパラメータを追加した。

- `browser_action`: ツールバー上に表示するアイコンなどのUIの設定
- `background`: アイコンがクリックされたときの動作を定義
- `permissions`: Chromeの様々なAPIを利用するための権限の設定（詳細は[Declare Permissions](https://developer.chrome.com/extensions/declare_permissions)を参照）

## コーディング

設定ファイルの準備が整ったので、続いて`manifest.json`の`background.scripts`で指定した`background.js`というファイルを`manifest.json`と同じディレクトリに作成し、コードを書いていく。

```js
// Extensionのアイコンがクリックされたときに呼ばれる
chrome.browserAction.onClicked.addListener(function() {
  // デバッグコンソールにメッセージを表示
  console.log('hello world');
});
```

コメントに記載の通り、ブラウザの右上に表示したアイコンをクリックすると、コンソールに`hello world`と表示されるプログラムを用意した。

## Extensionの読み込みと実行

Extensionは最終的に`.crx`という形式にパッケージングした状態でChromeのウェブストアにリリースされる。しかし、開発時に毎回その形式に変換するのは面倒なので、ローカルのChromeにパッケージングしていない状態のプログラムを読み込んでテストを行う。

- Chromeを起動し、URLバーから`chrome://extensions`を開く（または右上の設定メニューを開くボタンクリックし、Extensions > Toolsに移動する）。
- 右上のDeveloper modeのチェックボックスがオンになっていることを確認する。
- Load unpacked extension...をクリックし、`manifest.json`のあるディレクトリを選択して読み込む。

すると、ブラウザの右上に新しいアイコンが現れる。そのアイコンにマウスオーバーし、先ほど`manifest.json`の`browser_action.default_title`で定義した"Make this page red"という文字列が表示されれば、Extensionは正しく読み込まれている。

さらに`chrome://extensions`のページに戻りInspect views: background_pageというリンクをクリックすると、バックグラウンドで実行中のスクリプト用のデバッグコンソール別ウィンドウで表示される。そのコンソール上に"hello world"という文字列が表示されていれば、先ほどのコードも期待通りに動作していることがわかる。

## コーディングの続き

コンソールに文字列を出力していた部分を、実際にページを赤くするコード置き換えていく（Chrome Extensionで利用可能なAPIについては[Developer's Guide](https://developer.chrome.com/extensions/devguide)を参照）。

```js
// Extensionのアイコンがクリックされたときに呼ばれる
chrome.browserAction.onClicked.addListener(function(tab) {
  console.log('Turning ' + tab.url + ' red!');
  chrome.tabs.executeScript({
    code: 'document.body.style.backgroundColor="red"'
  });
});
```

もう一度`chrome://extensions`に戻ってReloadボタンをクリックし、適当なページを開いた状態でExtensionのアイコンをクリックすると、ページの背景が赤くなれば成功となる。

## リンク

参考にしたページの一覧はこちら。

- [Getting Started: Building a Chrome Extension](https://developer.chrome.com/extensions/getstarted) - 最も基本的なbrowser actionのチュートリアル
- [Overview](https://developer.chrome.com/extensions/overview) - browser action, page action, background, content scriptsなどのアーキテクチャの概要とChromeが提供するAPIについて
- [Developer's Guide](https://developer.chrome.com/extensions/devguide) - APIの詳細ページ
- [Sample Extensions](https://developer.chrome.com/extensions/samples) - サンプル一覧





