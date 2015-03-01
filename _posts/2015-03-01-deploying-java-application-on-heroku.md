---
layout: post
title: JavaアプリケーションをHerokuにデプロイする
summary: JavaでREST APIを作ったのでどこかにデプロイしてみたくなった。
date: 2015-03-01 23:54
---

## 背景

Javaで[Restletを使ったREST APIのサンプル][tatsuyaoiw/restlet-app]を書いていたら、どこかにデプロイしたくなったので[PaaSを色々検討した][Platform as a Service Provider Comparison]。その結果、過去にNodeのアプリのデプロイしたこともあるHerokuをJavaでも試しに使ってみることにした。

## 概要

基本的に、以下のドキュメントを一通り読んでいったらできた。

- [Getting Started with Java on Heroku]
- [Deploying Java Apps on Heroku]

## 必要なもの

- [Herokuアカウント][Heroku account]
- [Maven 3]

## 手順

### Heroku Toolbeltのインストール

最初に[Heroku Toolbelt]をインストールする。Heroku Toolbeltをインストールすると`heroku`コマンドや`foreman`コマンド（後述）が使えるようになる。以降の手順はほぼすべて`heroku`コマンドを使って行うことになる。

### Herokuへログイン

まずはHerokuにログインする。

```
$ heroku login
```

### Mavenプロジェクトの作成

次にMavenプロジェクトを用意する。既にあるものを使っても良いが、Herokuが提供しているサンプルもあるので、そっちを使っていく。

```
$ git clone https://github.com/heroku/java-getting-started.git
$ cd java-getting-started
```

### Procfileの作成

[Procfile][Process Types and the Procfile]はHeroku上で実行されるアプリケーション用の設定ファイルで、アプリケーションの起動時に実行されるコマンドを記載する。

```
$ web: java $JAVA_OPTS -cp target/classes:target/dependency/* Main
```

上記の`web`というのは[プロセスタイプ][Process Types and the Procfile]と呼ばれる宣言部分で、この`web`を使うとHTTPを利用してHeroku上のアプリケーションにアクセスできるようになる。

### Herokuアプリケーションの作成

続いてHeroku上にアプリケーションを作成する。

```
$ heroku create
Creating warm-eyrie-9006... done, stack is cedar-14
http://warm-eyrie-9006.herokuapp.com/ | https://git.heroku.com/warm-eyrie-9006.git
Git remote heroku added
```

`heroku create`を実行すると、適当なアプリケーション名 + herokuapp.com のドメインが振り分けられる（後から変更可能）。また、新たに`heroku`というGitのリモートレポジトリがローカルのGitプロジェクトに登録される。

### Herokuへデプロイ

アプリケーションのコードをデプロイする。

```
$ git push heroku master
```

`heroku`のリモートレポジトリにコードをPushすると、自動的にMavenのビルドが開始され、ビルドが正常に完了するとGitのコミットがリモートに反映される。

アプリケーションはURL（http://warm-eyrie-9006.herokuapp.com/ など）または以下のコマンドを使って参照
できる。

```
$ heroku open
```

### ログの確認

デプロイされたアプリケーションのログは、次のコマンドで確認できる。

```
$ heroku logs --tail
```

### foremanを使ったローカル実行

Heroku Toolbeltをインストールすると、`foreman`というコマンドが実行できるようになる。Foremanは`Procfile`を読み、Herokuへのデプロイをローカルで再現することができる。

```
$ foreman start web
```

上記のコマンドを実行すると、`http://localhost:5000`でアプリケーションが起動する。

## その他

### Maven dependency pluginの設定

[Deploying Java Apps on Heroku]を読んで`pom.xml`に設定しておく。

### ポート番号

Herokuの`web`プロセスタイプで使われるHTTPのポート番号は、環境変数`$PORT`から取得できる。Foreman実行時は`$PORT`の値は5000となる。

## サンプル

- [tatsuyaoiw/restlet-app]

[tatsuyaoiw/restlet-app]: https://github.com/tatsuyaoiw/restlet-app
[Platform as a Service Provider Comparison]: http://www.paasify.it/
[Getting Started with Java on Heroku]: https://devcenter.heroku.com/articles/getting-started-with-java
[Deploying Java Apps on Heroku]: https://devcenter.heroku.com/articles/deploying-java
[Heroku account]: https://signup.heroku.com/dc
[Maven 3]: http://maven.apache.org/
[Heroku Toolbelt]: https://toolbelt.heroku.com/
[Process Types and the Procfile]: https://devcenter.heroku.com/articles/procfile
