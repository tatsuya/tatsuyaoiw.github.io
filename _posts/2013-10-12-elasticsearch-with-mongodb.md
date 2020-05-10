---
layout: post
title: Elasticsearch + MongoDBを試す
date: 2013-10-22 06:44:00
tags: [Japanese]
---

ElasticsearchとMongoDBの組み合わせを試してみた。

## MongoDBのインストール

Homebrewからインストール。

    brew install mongodb

## MongoDBの起動

起動は `mongod` コマンドを実行。

    mongod

MongoDBの設定ファイルは `/usr/local/etc/mongod.conf` なので、デフォルトの設定を変えたいときはこれを編集するか、もしくはコマンドラインのオプションで上書きすることも可能。

## Elasticsearchのインストール

ソースダウンロード or Homebrewでインストール。

    brew install elasticsearch

## river-mongodbをインストール

続いてElasticsearchのMongoDB用riverプラグインである `elasticsearch-river-mongodb` をインストールする。インストールはElasticsearchのホームディレクトリから `plugin` コマンドを実行する。

    bin/plugin -i com.github.richardwilly98.elasticsearch/elasticsearch-river-mongodb/1.7.0

Elasticsearchを起動すると、インストールしたriverプラグインが読み込まれるようになる。（ログは見やすいように一部整形済み）

    $ ./bin/elasticsearch -f
    [INFO ][node   ] [Celestial Madonna] version[0.90.5], pid[7774]
    [INFO ][node   ] [Celestial Madonna] initializing ...
    [INFO ][plugins] [Celestial Madonna] loaded [mongodb-river], sites [river-mongodb]
    [INFO ][node   ] [Celestial Madonna] initialized
    [INFO ][node   ] [Celestial Madonna] starting ...
    ...

## river-mongodbの設定

次にriver-mongodbの設定を行う。

    {
      "type" : "mongodb",
      "mongodb" : {
        "servers" : [
          { "host" : "localhost", "port" : 27017 }
        ],
        "db" : "shelf",
        "collection" : "books"
      },
      "index" : {
        "name" : "books"
      }
    }

`type` プロパティの値にはriverの名前である"mongodb"を。`mongodb` プロパティの値はオブジェクトで、MongoDBのホスト名やポート番号、データの取得元となるデータベース名、コレクション名を指定。最後に `index` プロパティの値に、取得したデータの保存先となるインデックス名を指定した。

この設定を`config.json`という名前で適当な場所に保存しておく。

## MongoDBをReplicaSetとして起動する

river-mongogbは、MongoDBのインスタンスがReplicaSetとして起動していないとデータを読み取れないとのことなので、最初に実行したMongoDBインスタンスをいったんkillして、再度 `mongod`コマンドにオプションを加えて実行する。

    mkdir -p ~/sandbox/mongodb/data/1
    mongod --replSet rs0 --port 27017 --dbpath ~/sandbox/mongodb/data/1 --rest

`--replSet` オプションを追加すると、MongoDBはレプリカセットの実行モードで起動する。オプションの値には適当なレプリカセット名を入れている（ここでは `rs0` ）。また `--dbpath` にはデータの保存先のパスとして `~/sandbox/mongodb/data/1` を指定した（ここのパスは別途作っておくこと）。さらに `--rest` オプションを指定すると[http://localhost:28017/](http://localhost:28017/)から各種ステータスが見れるようになるので、付けておいたほうが。

次に `mongo` コンソールからレプリカセットの設定と初期化を行う。

    > config = {_id: 'rs0', members: [
    ... {_id: 0, host: '127.0.0.1:27017'}]
    ... }

`rs.initiate()` コマンドで初期化。

    > rs.initiate(config)
    {
      "info" : "Config now saved locally.  Should come online in about a minute.",
      "ok" : 1
    }

また、`rs.status()` で現在の設定を確認できる。

## river-mongodbの設定の反映

さきほどの `config.json` を、`/_river/<river名>/_meta` というエンドポイントに送信する。

    curl -XPUT 'localhost:9200/_river/mongotest/_meta' -d @config.json

設定が正しく反映されたかどうか `/_river/<river名>/_status` から確認できる。

    curl -XGET 'localhost:9200/_river/mongotest/_status?pretty'

以下のような結果が返る。

    {
      _index: "_river",
      _type: "mongotest",
      _id: "_status",
      _version: 2,
      exists: true,
      _source: {
        ok: true,
        node: {
          id: "qiz5CYTfRVCU0wxzHaQvbg",
          name: "Domino",
          transport_address: "inet[/192.168.11.2:9300]"
        }
      }
    }

## データの投入

riverが正しく動いているか、実際にデータを登録して試してみる。mongoコンソールを起動して、適当なサンプルを登録すると、

    rs0:PRIMARY> use shelf
    switched to db shelf
    rs0:PRIMARY> db.books.insert({ "title": "ElasticSearch Server", "price": 14.44 })

ほぼリアルタイムで、Elasticsearchのログにドキュメントのインデックスがされたことを示すメッセージが書き込まれる。

    [INFO ][cluster.metadata         ] [Domino] [_river] update_mapping [mongotest] (dynamic)
    [INFO ][org.elasticsearch.river.mongodb.MongoDBRiver$Indexer] Indexed 1 documents, 1 insertions, 0 updates, 0 deletions, 0 documents per second
    [INFO ][cluster.metadata         ] [Domino] [books] update_mapping [shelf] (dynamic)

検索してみると、

    curl -XGET 'http://localhost:9200/books/_search?pretty'

先ほど登録したドキュメントがヒットした。

    {
      "took" : 1,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
      },
      "hits" : {
        "total" : 1,
        "max_score" : 1.0,
        "hits" : [ {
          "_index" : "books",
          "_type" : "shelf",
          "_id" : "526cc5b872ab41c720792a10",
          "_score" : 1.0, "_source" : {"_id":"526cc5b872ab41c720792a10","title":"ElasticSearch Server","price":14.44}
        } ]
      }
    }