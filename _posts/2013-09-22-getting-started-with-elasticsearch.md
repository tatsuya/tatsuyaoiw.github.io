---
layout: post
title:  "elasticsearchで全文検索"
date:   2013-09-22 07:19:00
---

## インストール

elasticsearchをダウンロードしてインストール。

    wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-0.90.5.tar.gz
    tar zxvf elasticsearch-0.90.5.tar.gz
    cd elasticsearch-0.90.5
    bin/elasticsearch -f

## インデックスの作成

インデックスはcurlコマンドを使って作成できる。下の例では、twitterというインデックスのtweetタイプ（データベースでいうところのテーブル）にツイートを登録している。

    curl -XPUT localhost:9200/twitter/tweet/1 -d '
    {
    "user" : "tatsuyaoiw",
    "message" : "this is my first tweet!",
    "postDate" : "20130922T00:00:00"
    }'

1つだけだと寂しいのでもうひとつ。

    curl -XPUT localhost:9200/twitter/tweet/2 -d '
    {
    "user" : "tatsuyaoiw",
    "message" : "you know, for search",
    "postDate" : "20130922T07:00:00"
    }'

## インデックスの確認

最初のツイート。

    $ curl -XGET localhost:9200/twitter/tweet/1

    {"_index":"twitter","_type":"tweet","_id":"1","_version":1,"exists":true, "_source" : 
    {
    "user" : "tatsuyaoiw",
    "message" : "this is my first tweet!",
    "postDate" : "20130922T00:00:00"
    }}

2回目のツイート。

    $ curl -XGET localhost:9200/twitter/tweet/2

    {"_index":"twitter","_type":"tweet","_id":"2","_version":1,"exists":true, "_source" : 
    {
    "user" : "tatsuyaoiw",
    "message" : "you know, for search",
    "postDate" : "20130922T07:00:00"
    }}

ちなみに、インデックス時にユニークキーを自動で生成するには、PUTの代わりにPOSTを使う。

    $ curl -XPOST localhost:9200/twitter/tweet/ -d '
    {
    "user" : "tatsuyaoiw",
    "message" : "...",
    "postDate" : "20130922T08:00:00"
    }'

## インデックスの検索

インデックスが作成されていることを確認できたら、いよいよ検索してみる。tweetというキーワードを含むツイートを検索。

    curl -XGET localhost:9200/twitter/tweet/_search?q=message:tweet

別のキーワードでも。

    curl -XGET localhost:9200/twitter/tweet/_search?q=message:search

検索対象をmessageフィールドからuserフィールドに変更。

    curl -XGET localhost:9200/twitter/tweet/_search?q=user:tatsuyaoiw

また、elasticsearchはJSON形式でのクエリもサポートしていて、例えばSolrのRangeQueryのような複雑なパラメータは以下のとおり。

    curl -XGET localhost:9200/twitter/tweet/_search -d '
    { query: { range : { postDate : { from : "20130901", to : "20131001"}}}}}'

    curl -XGET localhost:9200/twitter/tweet/_search -d '
    { query: { range : { postDate : { from : "20130922T00:00", to : "20130922T01:00"}}}}}'