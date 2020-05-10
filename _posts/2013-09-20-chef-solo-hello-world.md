---
layout: post
title: ChefでHello, World!
date: 2013-09-20 10:58:00
author: Tatsuya Oiwa
tags: [Japanese]
---

[Chef](http://www.opscode.com/chef/ "Chef")はサーバの状態をコードに記述しておくことで、デプロイメントや設定の更新を自動化するツール。

## レポジトリ＞クックブック＞レシピ

Chefでは、コード化された手順をレシピ - Recipeと呼び、複数のレシピをまとめたものをクックブック - Cookbook、さらに複数のクックブックがまとまったセットをレポジトリ - Repositoryやキッチン - Kitchenと呼ぶ。

## Chefをはじめる

### レポジトリの作成

まずはレポジトリの作成から。OpsCodeがgithubに公開しているレポジトリのひな形をダウンロードする。

    $ git clone https://github.com/ospcode/chef-repo.git

## クックブックの作成

次にクックブックを作る。クックブックの作成にはknifeというコマンドを使う。

    $ knife configure

最初にknife configureで初期設定をする。色々と質問されるがすべてデフォルトでOK。初期化が完了すると~/.chef/knife.rbにknifeの設定ファイルが保存される。設定が終わったら、knifeコマンドで実際にクックブックを作成していく。

    $ cd chef-repo
    $ knife cookbook create hello -o cookbooks

cookbooksディレクトリ内にhelloというクックブックが作られた。

### レシピの編集

クックブックを作った時点でレシピのひな形が自動でできているのでそれを編集する。

    $ vi cookbooks/hello/recipes/default.rb

以下のようにログとして "Hello, Chef!" を出力するレシピを作成する。

    #
    # Cookbook Name:: hello
    # Recipe:: default
    #
    # Copyright 2013, YOUR_COMPANY_NAME
    #
    # All rights reserved - Do Not Redistribute
    #
    log "Hello, Chef!"

### Chef Soloの実行

Chef Solo実行にはchef-soloコマンドを実行するが、先に実行するレシピを記述したJSONファイルを用意しておく。適当にlocalhost.jsonという名前でファイルを作り、chef-repoディレクトリの直下に置く。

    // localhost.json
    {
        "run_list": [
            "recipe[hello]"
        ]
    }

次にChefが利用するテンポラリディレクトリやクックブックのパスを指定する設定ファイルを、solo.rbという名前で同じくchef-repoディレクトリ直下に置く。

    # solo.rb
    file_cache_path "/tmp/chef-solo"
    cookbook_path ["/home/tatsuyaoiw/sandbox/chef-repo/cookbooks"]

準備ができたら、作った二つの設定ファイルを指定して、chef-soloコマンドを実行する。Chef Soloはサーバーのあらゆるファイルを操作するため、実行にsudoが必要な場合もあるので注意。

    $ chef-solo -c solo.rb -j ./localhost.json

以下のように"Hello, Chef!"が出力されればOK。

    $ chef-solo -c solo.rb -j ./localhost.json 
    Starting Chef Client, version 11.6.0
    Compiling Cookbooks...
    Converging 1 resources
    Recipe: hello::default
      * log[Hello, Chef!] action write
    
    Chef Client finished, 1 resources updated
