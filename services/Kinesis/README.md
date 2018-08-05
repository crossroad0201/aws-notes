Kinesis
====

# Data Streams

* パーティションキーは、レコードの順序を保証したい最小単位で決める。
  * パーティションキーを分散させないと、シャードを増やしてもスケールしない。

* [シャードごとに時間単位で課金](https://aws.amazon.com/jp/kinesis/data-streams/pricing/)されるため、
常時データが入ってこない用途ではコストパフォーマンスが悪い。
  * AWSフルマネージドサービスのなかでも比較的コストが高く、無邪気にストリームを作っていくとつらい。
  * シャード数の増加、レコード保持期限の延長によってさらにコストが増える。
  * シャード追加によるスケーリングは最後の手段として、アプリケーションで可能な限り処理を効率化することを考える。

* `PutRecords` で複数レコードをまとめてPUTする場合、エラーをハンドリングするだけではなく、 [レスポンスの内容からすべてのレコードがPUTされたかどうか](https://docs.aws.amazon.com/ja_jp/streams/latest/dev/developing-producers-with-sdk.html#kinesis-using-sdk-java-putrecords) を確認する必要がある。
  * PUTに失敗したレコードを個別にリトライするなどのリカバリ処理を行う。
  * この場合、レコードのPUT順が意図したとおりにならない可能性があるので、レコードの順序保証が必要な場合は `PutRecords` を使用すべきではない。

* 厳密にレコードを順序通りにPUTしなければならない場合は、 [SequenceNumberForOrderingパラメタ](https://docs.aws.amazon.com/ja_jp/streams/latest/dev/developing-producers-with-sdk.html#kinesis-using-sdk-java-putrecord) を指定する必要がある。

## Lambdaのイベントソースとしての使用

* Lambda関数でエラーが発生すると、そのバッチ（取得したレコードのかたまり）全体がリトライされる。
  * ストリームのレコード保持期間の間、延々とリトライされて次のバッチに進まなくなるため、Lambda関数でエラーを返すのではなく、デッドレターキューに避けてスキップするなどの対策が必要。
  * 当然、スキップすれば順序性が保証されなくなるので、要件次第で検討する。

* 1つのストリームに複数のLambda関数を関連付けると遅延が発生するため、原則としてLambda関数は1つだけしか付けることができない。
  * ネット上にある情報でも、1ストリームに複数のLambda関数を付けるパターンがいくつも紹介されているが、実は落とし穴。
  * Lambda関数はストリームに新しいレコードがないかを定期的にポーリング（秒間2回程度）するが、
    Kinesisストリームには [読み取りは最大 1 秒あたり 5 件のトランザクション](https://docs.aws.amazon.com/ja_jp/streams/latest/dev/service-sizes-and-limits.html) と言う制限があるため、Lambda関数を2-3個付けるとこの制限を超えてしまう。
  * 制限を超えると [ReadProvisionedThroughputExceeded](https://docs.aws.amazon.com/ja_jp/streams/latest/dev/monitoring-with-cloudwatch.html) メトリクスが上昇する。このメトリクスは通常 `0` で推移するのが正常。
  * Lambda関数のイベントソース設定で、ポーリング頻度を調節できればいいのだが...

## 運用

* `UpdateShardCount`  によるシャード数の変更には [回数制限](https://docs.aws.amazon.com/ja_jp/streams/latest/dev/service-sizes-and-limits.html) がある。
  * リシャーディング（`SplitShard`, `MergeShard`）にはこのような制限はない。

* シャード数を変更すると、既存のすべてのシャードはCLOSEされて、新しくシャードが作成される。

* リシャーディングすると、対象のシャードはCLOSEされて、新しくシャードが作成される。

* CLOSEされたシャードにもレコードが残っているため、レコードの順序を保証したい場合はまずCLOSEされたシャードのレコードを処理しきってから、新しいシャードのレコードを処理しないと、処理順序が入れ替わる可能性がある。
  * 原則として、 ~~シャード数変更／~~ リシャーディングする際は、レコードの流入元のアプリケーションを停止してストリームに溜まっているレコードをすべて処理しきってから実施するのが確実。
  * シャード数変更した場合は、CLOSEされたシャードに残っているレコードがすべて処理されきってから、新しいシャードの処理がはじまることに注意。（リシャーディング時も同じかどうかは未確認）
  そのため、トラフィックをさばききれなくなってからシャード数を増やしてスケールさせても、すでに溜まっているレコードが早く処理されるわけではない。

* シーケンスNoは、シャード数変更／リシャーディングしても必ず大きな値になっていくことが保証される。
  * シーケンスNoの採番仕様は公開されていないが、シャードIDに応じた値＋時系列 になっているようだ。

# リンク集

## AWS公式

* [開発者ガイド](https://docs.aws.amazon.com/ja_jp/streams/latest/dev/introduction.html)
