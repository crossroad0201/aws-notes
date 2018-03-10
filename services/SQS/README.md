SQS
====

# 全般

* [ReceiveMessage API](https://docs.aws.amazon.com/ja_jp/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues.html#standard-queues-message-sample) はキューに溜まっているメッセージをすべて返してくれるわけではない。（たとえば、10メッセージ溜まっていても、2メッセージとか3メッセージしか取れないことがある）


# エラー処理

* 正常に処理できずにキューから削除できなかったメッセージはそのままだと何度も何度も取得されてしまうので、[デッドレターキュー](https://docs.aws.amazon.com/ja_jp/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html) を設定して、所定の回数処理できなかったメッセージを退避させる。

* エラーなどでメッセージを削除しなかった場合、そのメッセージはキューに設定されたデフォルトの可視性タイムアウトが経過するまで再受信（リトライ）されないが、[ChangeMessageVisibility API](https://docs.aws.amazon.com/ja_jp/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html#changing-message-visibility-timeout) で当該メッセージの可視性タイムアウトを短く変更することで、すぐに再受信（リトライ）できるようになる。
