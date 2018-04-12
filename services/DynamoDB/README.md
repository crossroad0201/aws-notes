Dynamo DB
====

# 全般

* RDBと [用語](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html) が違う。

||表|行|列|
|----|----|----|----|
|*DynamoDB*|テーブル(Table)|項目(Item)|属性(Attribute)|
|*RDB*|テーブル|レコード|カラム|

* 高機能な一覧系の要件がある場合は向いていない。
  * LSI、GSIの数に制限があり、多様な一覧取得はできない。
  * nページ目からm件取得、のようなページングが苦手。
  * ソートは、レンジキーによる昇順／降順しか指定できない。

* ハッシュキー単独、またはハッシュキー+レンジキー で項目が一意にならなければならない。

* `condition-expression` を使って [楽観ロック](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/Expressions.ConditionExpressions.html#Expressions.ConditionExpressions.SimpleComparisons) できる。

* バックアップ／リストアが難しい。
  * [オンデマンドバックアップ](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/BackupRestore.html)

# Local Secondary Index

* テーブル作成と同時にしか作成できない。（後から追加はできない）

* ハッシュキーにはテーブルと同じ属性を指定しなければならない。

* レンジキーの値は重複しても構わない。（テーブルと異なり、ハッシュキー+レンジキー で項目が一意にならなくても良い）

* レンジキーの属性がなくてもかまわない。
  * レンジキーの属性がない項目は、当該LSIを使った検索でHITしなくなる。

# Global Secondary Index

* テーブルとは別にキャパシティを消費する。
  * オートスケールの設定も別途必要。

# オートスケーリング

* プロビジョニング済みキャパシティの超過を検知してから、実際にキャパシティがスケールされるまで数分程度のタイムラグがある。
  * キャパシティ超過のエラーを受けてリトライしても、オートスケーリングが間に合わないことが多い。

* オートスケーリング設定を作成する際、秒間トランザクションに制限があるので、 *CloudFormation* などで一気にテーブル・オートスケーリング設定を作成する際は注意が必要。
  * 制限緩和依頼を出す。
  * *CloudFormation* テンプレートで `DependsOn` を指定してオートスケーリング設定が同時並行で作成されないようにする。
