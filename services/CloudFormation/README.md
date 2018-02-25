Cloud Formation
====

# 全般

* テンプレートは YAML で書く。
  * JSONはつらい。（煩雑、コメントが書けないなど）

* AWSコンソールによるGUI操作による運用は煩雑になる`ので、CLIから操作できる何らかのツール／フレームワークの使用が望ましい。
  * [Serverless Framework](https://serverless.com)
  * [fabricawscfn](https://github.com/crossroad0201/fabric-aws-cloudformation)
  * etc

* スタックをどのような粒度にするかは、運用を想定して決める。
  * 常に同じタイミングで作成／変更／削除するリソースは、1つのスタックにまとめる。
  * 個々のタイミングで作成／変更／削除したいリソースは、単独のスタックにしておく。

* 条件によってリソースの作成する／しないを切り替えたい場合は、 [Condition](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html) を使う。
  * `Outputs` でリソースを出力している場合は、`Outputs` にも `Condition` を指定する必要あり。

