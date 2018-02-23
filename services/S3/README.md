S3
====

# 全般

* オブジェクトのキー名は、ランダムな文字列（3-4文字）ではじまるようにする。
  * 同じ文字列ではじまると、同じパーティションに格納されるために [性能上の問題](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/request-rate-perf-considerations.html) になることがある。
  

