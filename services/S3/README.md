S3
====

# 全般

* ~~オブジェクトのキー名は、ランダムな文字列（3-4文字）ではじまるようにする。~~
  * ~~同じ文字列ではじまると、同じパーティションに格納されるために [性能上の問題](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/request-rate-perf-considerations.html) になることがある。~~
  * 2018/07 オブジェクトのキー名は連続していてもパフォーマンスが悪化しなくなった。 [参考](https://dev.classmethod.jp/cloud/aws/amazon-s3-announces-increased-request-rate-performance/)

* 同じバケットへのアクセスが多い場合(?)に、S3オブジェクトの読み取りエラーが発生することがままあり、リトライすればほぼ確実に成功するので、自動的にリトライするようにしておいたほうが良い。
  ```
  org.apache.http.ConnectionClosedException: Premature end of Content-Length delimited message body (expected: 64831; received: 53583)
      org.apache.http.impl.io.ContentLengthInputStream.read(ContentLengthInputStream.java:140),
      org.apache.http.conn.EofSensorInputStream.read(EofSensorInputStream.java:118),
      com.amazonaws.internal.SdkFilterInputStream.read(SdkFilterInputStream.java:66),
      com.amazonaws.event.ProgressInputStream.read(ProgressInputStream.java:159),
      com.amazonaws.internal.SdkFilterInputStream.read(SdkFilterInputStream.java:66),
      com.amazonaws.services.s3.model.S3ObjectInputStream.read(S3ObjectInputStream.java:135),
      com.amazonaws.internal.SdkFilterInputStream.read(SdkFilterInputStream.java:66),
      com.amazonaws.internal.SdkFilterInputStream.read(SdkFilterInputStream.java:66),
      com.amazonaws.event.ProgressInputStream.read(ProgressInputStream.java:159),
      java.security.DigestInputStream.read(DigestInputStream.java:124),
      com.amazonaws.services.s3.internal.DigestValidationInputStream.read(DigestValidationInputStream.java:47),
      com.amazonaws.internal.SdkFilterInputStream.read(SdkFilterInputStream.java:66),
      com.amazonaws.services.s3.model.S3ObjectInputStream.read(S3ObjectInputStream.java:135),
       :
  ```

# リンク集

## AWS公式

* [開発者ガイド](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/Welcome.html)


