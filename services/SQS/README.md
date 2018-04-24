SQS
====

# 全般

* 標準キューは順序保証されない。だいたいは送信した順に並ぶ、と言うこともなさそうなのでまったくバラバラの順序になると考えていい。

* [ReceiveMessage API](https://docs.aws.amazon.com/ja_jp/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues.html#standard-queues-message-sample) はキューに溜まっているメッセージをすべて返してくれるわけではない。（たとえば、10メッセージ溜まっていても、2メッセージとか3メッセージしか取れないことがある）

* キューの可視性タイムアウトは、そのメッセージを受信して実行される処理に要する時間を考慮して決定する。
処理時間が可視性タイムアウトを超えてしまうと、そのメッセージは削除できなくなる。（削除しようとすると `The receipt handle has expired` エラーが発生する）

# FIFOキュー

* 一部のリージョンでしか使用できない。

* 1度に受信できるメッセージ数など、標準キューと比較していろいろな制約があり、スループットも出ないし、スケールもしづらい。

* メッセージ送信時に [メッセージグループID](https://docs.aws.amazon.com/ja_jp/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html#FIFO-queues-understanding-logic) を指定する必要がある。
  * この メッセージグループID 単位に順序保証される。
  * 受信側は *メッセージグループID* を指定することはできない。メッセージ受信時には同じメッセージグループIDのメッセージが優先して取得される。
  * メッセージグループID は、[メッセージ属性にセットされている] (https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc//com/amazonaws/services/sqs/model/MessageSystemAttributeName.html) はず。

* メッセージは順序どおりにしか削除できない。

# エラー処理

* 正常に処理できずにキューから削除できなかったメッセージはそのままだと何度も何度も取得されてしまうので、[デッドレターキュー](https://docs.aws.amazon.com/ja_jp/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html) を設定して、所定の回数処理できなかったメッセージを退避させる。

* エラーなどでメッセージを削除しなかった場合、そのメッセージはキューに設定されたデフォルトの可視性タイムアウトが経過するまで再受信（リトライ）されないが、[ChangeMessageVisibility API](https://docs.aws.amazon.com/ja_jp/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html#changing-message-visibility-timeout) で当該メッセージの可視性タイムアウトを短く変更することで、すぐに再受信（リトライ）できるようになる。

# リンク集

## AWS公式

* [開発者ガイド](https://docs.aws.amazon.com/ja_jp/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
