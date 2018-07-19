VPC
====

# 全般

* マイクロサービスアーキテクチャを採用する場合でも、サービスごとに独立した *VPC* を構成するのではなく、サービス全体で1つの *VPC* を共用したほうが良い。
  * *ECS* 上でサービスを稼働させる場合に、必要に応じて物理サーバを共用することができる。（ランニングコストの削減）
  * *RDS* などのデータストレージを、必要に応じて共用することができる。(ランニングコストの削減）
  * *DynamoDB* を利用する場合に、 *DAX* を利用できる。
  