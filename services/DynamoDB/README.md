Dynamo DB
====

# 全般

* RDBと [用語](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html) が違う。

||表|行|列|
|----|----|----|----|
|*DynamoDB*|テーブル(Table)|項目(Item)|属性(Attribute)|
|*RDB*|テーブル|レコード|カラム|

* 高機能な一覧・検索の要件がある場合は向いていない。
  * LSI、GSIの数に制限があり、多様な一覧取得はできない。
  * nページ目からm件取得、のようなページングが苦手。
  * ソートは、レンジキーによる昇順／降順しか指定できない。
  * 高機能な一覧・検索の要件がある場合は、[CQRSパターン](https://docs.microsoft.com/ja-jp/azure/architecture/patterns/cqrs) の採用を検討する。
    * 例えば、読み取り専用モデルを *ElasticSearch Service* で構築し、*DynamoDB Streams* を使って変更されたデータを *ElasticSearch* に反映させる。
    * 当然、読み取り専用モデルには非同期に反映されるため、結果整合性になることに注意。

* ハッシュキー単独、またはハッシュキー+レンジキー で項目が一意にならなければならない。

* `condition-expression` を使って [楽観ロック](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/Expressions.ConditionExpressions.html#Expressions.ConditionExpressions.SimpleComparisons) できる。

* バックアップ／リストアが難しい。
  * [オンデマンドバックアップ](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/BackupRestore.html)

* AWS公式の [DynamoDB のベストプラクティス](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/best-practices.html) を確認する。

# Local Secondary Index

* テーブル作成と同時にしか作成できない。（後から追加はできない）

* ハッシュキーにはテーブルと同じ属性を指定しなければならない。

* レンジキーの値は重複しても構わない。（テーブルと異なり、ハッシュキー+レンジキー で項目が一意にならなくても良い）

* レンジキーの属性がなくてもかまわない。
  * レンジキーの属性がない項目は、当該LSIを使った検索でHITしなくなる。

# Global Secondary Index

* テーブルとは別にキャパシティを消費する。
  * オートスケールの設定も別途必要。

# クエリ

* テーブルおよびインデックスのキー属性（ハッシュキー、レンジキー）に対しては、[キー条件式](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/Query.html#Query.KeyConditionExpressions) であげられている式しか使えない。（IN句や、ORなどの論理演算式も使えない）

# オートスケーリング

* プロビジョニング済みキャパシティの超過を検知してから、実際にキャパシティがスケールされるまで数分程度のタイムラグがある。
  * キャパシティ超過のエラーを受けてリトライしても、オートスケーリングが間に合わないことが多い。

* オートスケーリング設定を作成する際、秒間トランザクションに制限があるので、 *CloudFormation* などで一気にテーブル・オートスケーリング設定を作成する際は注意が必要。
  * 制限緩和依頼を出す。
  * *CloudFormation* テンプレートで `DependsOn` を指定してオートスケーリング設定が同時並行で作成されないようにする。

# DynamoDBストリーム

* DynamoDBストリームのシャード数はテーブルのパーティション数によって決まるので、ハッシュキーが分散していればしているほどパーティションが分散してシャード数が多くなり、処理効率が高くなる。

* トリガーとして設定できる *Lambda* は、基本的に1つだけ。
  * *Kinesis* と同様、[内部的にはストリームをポーリングする](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/with-ddb.html)ため、*Lambba* を複数設定するとストリームの [読み取りでスロットリングが発生](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/Limits.html#limits-dynamodb-streams) する。

# リンク集

## AWS公式

* [開発者ガイド](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/Introduction.html)
