---
layout: post
title: 「SQLアンチパターン」読んだ
date: 2019-03-02 10:58
published: true
---

[SQLアンチパターン](https://www.oreilly.co.jp/books/9784873115894/)読みながら[Twitterでつぶやいてた](https://twitter.com/tatsuyaoiw/status/1086442006497247232
)のを雑にまとめます。

SQLの正式な発音は「エス・キュー・エル」です。口語的に「シークェル」と発音されることもあり（略）

1章読み始めたけど、いきなりアカウントIDをカンマ区切りでカラムに入れようとしててアンチパターンすぎて笑った。が、Intersection Tableで解決というのは実務やりながら知ったから、逆にその経験がなくてゼロから設計しろとなったらカンマ区切りしてたかもしれないし笑えない。

2章、Tree構造のテーブル設計、学びが多い。Naive Tree vs Path Enumeration vs Nested Set vs Closure Tableの長所および短所。これは絶対知らないとできない。Nested Setなんかは背景にあるデータ構造知らないと、テーブル見ただけではTreeであることすら気づけなさそう。

3章、ID Required、賛否両論ありそう。

4章、Foreign Key Constraintちゃんとつけよう+Cascade Updateも設定しようっていう話。

5章、EAV（Entity Attribute Value）は悪であるということがわかった。解決策はSingle Table Inheritance or Concrete Table Inheritance or Class Table InheritanceだけどClass Table Inheritanceほぼ一択な予感。KVSかJSONなどのSerialized LOB（Large OBject）使えとも書いてある。

6章、Polymorphic Associations、学びが多い。一貫してRelational Databaseがサポートする制約を使ってデータ整合性を保つことの大切さを説いている。

7章、Multicolumn Attributes、tag_1 tag_2 tag_3みたいなのはDependent Tableで解決。

8章、ハッシュ値などでテーブル分割するHorizontal Partitioning、容量の大きなデータや可変長データを格納するカラムを別テーブルに分けるVertical Partitioningなどの話。

9章、Rounding Errorの問題を避けるために基本FLOAT型など二進数の有限精度で浮動小数点数を表すデータ型は使わない。

10章、BugStatusなどのごく少数の値だけをサポートしたいカラムに対しては、ENUMやCHECK制約、DOMAINやUDTよりも参照テーブルを使ったほうが拡張性が高くて良い。

11章、画像データは一般的にBLOB型でカラムに保存するのではなく、外部ファイルに画像を保存しておいてファイルパスをVARCHAR型などで入れるケースが多いが、こうしてしまうとトランザクション管理ができずに参照整合性を保てなったり、バックアップを別々に取らなければいけないといった問題がある。そのためBLOB形でそのままカラムに入れてしまうという手も十分に検討する必要がある。

12章、闇雲にインデックスを使わない。

13章、NULLの取り扱いに注意。

14章、GROUP BYで陥りやすい罠の話。

15章、RAND関数使うとテーブルスキャン発生するから先にKey List取ってアプリケーションレイヤーでRandom Selectするとか工夫しようという話。

16章、全文検索するならパターンマッチはやめて全文検索エンジン使おうって話。まさかこの本でLucene/Solrに言及されるとは...Inverted Indexをテーブルで自作するって方法も紹介されてはいるけど現実問題これもアンチパターンだろうなあ。

17章、複雑なクエリは分割せよ。
18章、ワイルドカードでImplicitにカラム取得するときは要注意。

19章、パスワードを平文で保存するっていう最悪のアンチパターンについてだけど、じゃあどう実装したらいいのってところが詳細に書かれててよかった。ソルト付きパスワードハッシュをアプリケーションコードで計算して格納、が正解なんだけど、加えてパスワードリセットをどうするのとかも書かれてる。

20章、SQLインジェクションとPrepared Statement使おうって話。ORMとか使ってるとあんまり意識しなくなってしまうけど、大事だなあ。

21章、Psedukeyの欠番は埋めない。これ考えたこともなかったけど、確かにSequential IDに意味は無いというのを受け入れられない人はいるかも知れないので覚えておこう。

22章、SQLのデバッグは戻り値を見る、エラーを読むっていう至極当然な話だった...

23章、VCS使う、ドキュメント書く、テストする...

24章、突然ソフトウェア設計の話。モデルがActive Recordそのものである場合、ドメインモデル貧血症: Anemic Domain Modelを引き起こすと。その結果複数のコントローラー内でロジックが重複したり、モデル自身の振る舞いのテストをデータベースから切り離して行うことができないなどの問題が発生する。Active Recordの使用はプロトタイプ作成において効果を発揮するが、同時に技術的負債を生み出すのでリファクタリングちゃんとしろよって書いてある。

そしてDomain Driven Designの話。Active RecordのようなDAOとモデルとの関係はis-aではなくhas-aであるべき。しかしActive Recordを使用するほとんどのフレームワークはis-aを想定している。そこでDomain Modelの使用によってデータベース構造とモデルを分離して疎結合化、カプセル化することが重要。

まさかSQLアンチパターンでDDDの話になると思ってなかった。

25章、運用の話。ベンチマーク、テスト、バックアップ、例外処理、High Availability、Disaster Recoveryなど。

## まとめ

2章、6章、10章、24章あたりが特に参考になった。

