S3
====

# 全般

* ~~オブジェクトのキー名は、ランダムな文字列（3-4文字）ではじまるようにする。~~
  * ~~同じ文字列ではじまると、同じパーティションに格納されるために [性能上の問題](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/request-rate-perf-considerations.html) になることがある。~~
  * 2018/07 オブジェクトのキー名は連続していてもパフォーマンスが悪化しなくなった。 [参考](https://dev.classmethod.jp/cloud/aws/amazon-s3-announces-increased-request-rate-performance/)

  

# リンク集

## AWS公式

* [開発者ガイド](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/Welcome.html)


