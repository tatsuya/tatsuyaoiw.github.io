---
layout: post
title: Node.jsとRedisで作るTwitterクローン
date: 2015-06-09 09:29
author: Tatsuya Oiwa
tags: [Japanese]
---

## 背景

[Node.js in Actionの第9章 - Advanced Express](http://www.manning.com/cantelon/)を読みながら、[shoutbox](http://en.wikipedia.org/wiki/Shoutbox)というアプリケーションを作った。shoutboxは、ユーザ登録、ログイン、メッセージ投稿といった機能のみを持つシンプルなアプリケーションで、ひと通り実装し終わった頃、「これ、もうちょっと頑張ったらTwitterっぽくできるんじゃね」と思い、だいたい次のような機能を追加していった。

- フォロー/フォロワー
- 自分のツイートの一覧（user's timeline）
- 自分がフォローしているユーザのツイートの一覧（home timeline)

で、最終的にできたのがこれ: [Demo](https://twitterlikeapp.herokuapp.com/)

コードはこちら: [Github](https://github.com/tatsuyaoiw/twitter)

本ブログ記事では、上記のようなアプリケーションをどのように実装したのか、少し細かく書き残していこうと思う。

## 使用した技術

サーバーは[Node.js](https://nodejs.org/)で、Webアプリケーションフレームワークの[Express](http://expressjs.com/)を使用した。データストアはshoutboxの例で使われていた[Redis](http://redis.io/)をそのまま使用。HTMLの生成はテンプレートエンジンの[Jade](http://jade-lang.com/)を使っている。クライアントサイドのJavaScriptは極力使わないようにし（というか[Bootstrap](http://getbootstrap.com/)以外は無し）、ユーザーが何かアクションを起こす度に毎回サーバー側で新しくHTMLを生成するようにした。

## ユーザー登録

まずはじめに、ユーザー登録、ログインなどの認証機能を実装する。

各ユーザのIDは、Redisの[INCR](http://redis.io/commands/INCR)コマンドを使って生成した。

```
INCR user:ids
```

上記のIDを元に各ユーザーのオブジェクトを生成し、Redisの[Hash](http://redis.io/topics/data-types)に保存する。ユーザーオブジェクトのフィールド定義は以下の通り。

```
name (ユーザー名)
pass（暗号化したパスワード）
fullname（ユーザーのフルネーム）
```

例えば、ユーザー（ID:123）を新たに登録する場合、次のような実装となる。

```
INCR user:ids => 123
HMSET user:123 name "bob" pass "asdfjkl;" fullname "Bob Marley"
```

[HMSET](http://redis.io/commands/hmset)コマンドを使い、`user:123`というキーに対して、Hashオブジェクトを値として登録している。

## フォロー/フォロワー

次に、ユーザーのフォローおよびフォロワーの機能を実装する。フォロー/フォロワーの情報は、Redisの[Sorted Set](http://redis.io/topics/data-types)を使用し、フォロー時のタイムスタンプをスコアに利用することで時系列順にソートできるようにした。

```
user:123:followers => Sorted Set: ユーザーのフォロワーのID一覧
user:123:followings => Sorted Set: ユーザーがフォローしているIDの一覧
```

ユーザー（ID:123）を別のユーザー（ID:456）がフォローする際の実装は以下のようになる。

```
ZADD user:123:followers 1401267618 456
ZADD user:456:followings 1401267618 123
```

`1401267618` というのはフォローしたタイミングで取得したUnixタイムスタンプで、この値がそのままソート時のスコアとなる。

例えば、直近のフォロワー5人のIDを取得するには、[ZREVRANGE](http://redis.io/commands/zrevrange)コマンドを使うと良い。

```
ZREVRANGE user:123:followers 0 4
```

上記のコマンドは、ユーザー（ID:123）のフォロワーのリストのうち、スコアの高い順（時系列順の最新）にインデックスの`0`から`4`までの値を取得している。

## ツイート

ツイートもユーザと同様、[INCR](http://redis.io/commands/INCR)を使ってユニークIDを生成した後、各ツイートをHashとして保存する。

```
INCR tweet:ids => 1
```

ツイートオフジェクトのフィールド一覧。

```
text（ツイートのテキスト）
created_at（ツイートした時間）
user_id (ツイートしたユーザーのID)
```

ツイートの投稿。

```
HMSET tweet:1 text "Peace and Love" created_at 1401267618 user_id 123
```

## タイムライン

タイムラインは、次の2種類を別々に実装した。

- [user_timeline](https://dev.twitter.com/rest/reference/get/statuses/user_timeline) - ユーザーのツイート一覧
- [home_timeline](https://dev.twitter.com/rest/reference/get/statuses/home_timeline) - ユーザーがフォローしているユーザーのツイート一覧

各タイムラインはそれぞれ時系列順に並べられるため、フォロー/フォロワーで使用したのと同じく[Sorted Set](http://redis.io/topics/data-types)を使用する。

```
user:123:user_timeline => Sorted Set: ユーザーのツイートIDのリスト
user:123:home_timeline => Sorted Set: ユーザーがフォローしているユーザのツイートIDのリスト
```

あるユーザーがツイートを投稿すると、まずそのユーザーの`user_timeline`にツイートIDが追加される。続いて、そのユーザー自身の`home_timeline`にツイートIDを登録した後、さらにそのユーザーをフォローしているユーザーの`home_timeline`にも、同じツイートIDを登録する。

```
ZADD user:123:user_timeline 1401267618 1
ZADD user:123:home_timeline 1401267618 1
ZADD user:456:home_timeline 1401267618 1
...
```

なお、この処理はフォロワーの数によってパフォーマンスに与える影響が大きく変わる可能性があるので注意が必要となる。例えば、あるユーザーのフォロワーが100,000人いたとして、そのユーザーがツイートすると、1 + 100,000回のWrite処理が実行されることになる。この辺りのチューニングは今回の実装では全く行っていないので、もしユーザー劇的に増えるようなことがあれば対応しなくてはならない。

さて、タイムラインの取得は、フォロー/フォロワーと同じく[ZREVRANGE](http://redis.io/commands/zrevrange)コマンドを使えばよい。最新の50ツイートを取得する場合、以下のような実装となる。

```
ZREVRANGE user:123:home_timeline 0 49
```

## アプリケーションのデプロイ

デプロイ先には[Heroku](https://www.heroku.com/)を利用した。[Getting Started with Node.js on Heroku](https://devcenter.heroku.com/articles/getting-started-with-nodejs#introduction)というチュートリアルがあるので詳細な説明は省くが、ざっくり以下の手順でデプロイできる。

`Procfile`の作成。

```
web: npm start
```

Herokuにログインし、アプリケーションを作成してデプロイ。

```
$ heroku login
$ heroku create
$ git push heroku master
```

## 参考

いくつか同じような実装を見つけたので参考にさせてもらった。

- [antirez/retwis](https://github.com/antirez/retwis) - PHPとRedisを使ったTwitterのクローン
- [twissandra/twissandra](https://github.com/twissandra/twissandra) - PythonとCassandraを使ったTwitterのクローン

特に、[Redisのサイトで紹介されているRetwisのチュートリアル](http://redis.io/topics/twitter-clone)はわかりやすくてとても参考になった。
