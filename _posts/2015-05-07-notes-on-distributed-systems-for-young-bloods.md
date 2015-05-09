---
layout: post
title: 「Notes on Distributed Systems for Young Bloods」読んだ
summary: 分散システムを勉強中
date: 2015-05-07 18:48
---

[Jeff Hodges](http://www.somethingsimilar.com/about/)の[Notes on Distributed Systems for Young Bloods](http://www.somethingsimilar.com/2013/01/14/notes-on-distributed-systems-for-young-bloods/)というブログ記事を読んだのでそのメモ。

## Distributed systems are different because they fail often.

分散システムが他のシステムと大きく異なるのは、障害発生率の高さである。システム全体、またはシステムの一部の障害にどのように対処すべきかを適切に設計、実装しなければならない。

## Writing robust distributed systems costs more than writing robust single-machine systems.

分散システムはお金がかかる。なぜなら、分散システムにおける障害は、大容量データの処理中に発生したり、ネットワーク障害に起因するものなど様々で、これらは単一のマシンで再現することが難しいためだ。

## Robust, open source distributed systems are much less common than robust, single-machine systems. 

分散システムは多くのマシンを必要とするので、オープンソースのソフトウェアで十分にテストされているものは比較的少ないと考えるべき。

## Coordination is very hard.

複数のマシン間のコミュニケーションと、コミュニケーションに必要なコンセンサスは最小限に。

## If you can fit your problem in memory, it’s probably trivial.

複数のスイッチを跨いだシステムのデバッグは大変。単一のマシンで利用できるアルゴリズムは山ほどあるが、分散システムに適用できるものはそれほど多くない。

## “It’s slow” is the hardest problem you’ll ever debug.

「なんか遅い」の原因を見つけ出すのは骨の折れる作業。

## Implement backpressure throughout your system.

バックプレッシャーを実装すること。システムの一部が高負荷になったときに、それ以上をリクエストを受け付けないようにするなど対策しなければならない。

## Find ways to be partially available.

例えば、検索エンジンの場合、タイムアウトの上限を越えた場合、検索結果の一部分のユーザに返すことでシステムが応答しないという最悪の状況を避ける。これは、完全な検索結果を返すという品質とのトレードオフ。

## Metrics are the only way to get your job done. 

メトリクスを提供すること。ログファイルはデバッグに役立つこともあるが、多くの場合、大量のログ出力によって重要なメッセージを見逃してしまう。

## Use percentiles, not averages. 

平均ではなくパーセントを使う。平均はベルカーブ（正規分布）のデータには効果的だが、分散システムでそうなることはまれである。

## Learn to estimate your capacity.

必要なタスクを実行するのに必要なマシンの数は？必要な期間は？正しく見積もること。

## Feature flags are how infrastructure is rolled out.

新機能のリリースやバックエンドシステムのリリースを容易にするために効果的に用いること。

## Choose id spaces wisely.

単純なハッシュ値よりも、意味付けされたIDの方が最適な場合もある。例えば、IDによってデータのパーティショニングを効果的に実装するとレイテンシを下げることができる。

## Exploit data-locality.

データ処理はできるだけローカルで。ネットワークを跨ぐと時間がかかったりトラブルの元になる。また短時間で重い処理が続く場合は、それらの処理を一つにまとめるなどして最適化することができる。

## Writing cached data back to persistent storage is bad. 

キャッシュデータを永続化ストレージに戻すと思わぬデータ不整合を起こすことがあるので注意。

## Computers can do more than you think they can.

比較的複雑なCRUDオペレーションでも、秒間数千リクエストを数百ミリ秒以内に処理することができるくらい最近のマシンは高速なので、できるだけ単一のマシンで処理すること。

## Use the CAP theorem to critique systems.

分散システムの設計や評価にはCAP定理が便利。

## Extract services.

小さな単位のサービスに分割してリクエスト/レスポンススタイルのAPIで疎結合にすること。サービス間のコミュニケーションをライブラリで実装することで、アップグレードを容易にできる。

