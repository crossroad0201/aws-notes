Step Functions
====

# 全般

* ステートマシンには[わかりやすい名前](https://docs.aws.amazon.com/cli/latest/reference/stepfunctions/create-state-machine.html)を付ける。
  * デフォルトだと無為な名前になるので、 *Step Functions* のコンソール上で見たときに何がなにかわからなくなる。


* ステートマシンの起動時には、明示的にわかりやすい[名前](https://docs.aws.amazon.com/cli/latest/reference/stepfunctions/start-execution.html)を指定する。
  * デフォルトだと UUID になるので、 *Step Functions* のコンソール上で見たときに何がなにかわからなくなる。

* ステートマシンの起動は、非同期しかできない。（ステートマシンの終了を同期的に待つことはできない）
  * ステートマシンの終了を知りたい場合は、自前で（*DynamoDB* に記録するなどして）状態管理する必要がある。

* 1秒あたりのステートマシンの起動数には制限がある。（デフォルトだと [25回/秒](https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/limits.html#service-limits-api-action-throttling)）
  * 必要であれば制限緩和を申請する。ただし、比較的申請が通りにくく対応にもやや時間がかかる上に、上限が低い（実績で 200 が限度）
  * コスト面でも、恒常的にトラフィックが発生するような用途には向かない。
# コスト

* ステートマシン内の [状態遷移ごとに課金](https://aws.amazon.com/jp/step-functions/pricing/) されるため、トランザクション量が多い場合は *EC2* などのサーバーを立てたほうが安くなる。

# ステートマシンの記述

* ステートマシンはAWS公式にはJSON形式しかサポートしていないが、メンテナンスが大変なのでサードパティのフレームワーク（[Serverless Framework](https://github.com/horike37/serverless-step-functions)）を利用するなどして、YAMLで記述するようにする。

* ステートの名前などには[最大80文字の制限があるので、80文字以内になるように命名ルールを決める。
（[制限](https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/limits.html#service-limits-general)にはステートマシンの名前しか触れられていないが）

* ステートの名前は、ステートマシン全体でユニークでなければならない。（ネストしたステート内、のようなスコープの概念はない）

* ステートマシンからの *Lambda* の呼び出し時には、AWSのインフラ都合による瞬断

* ステートマシンからの *Lambda* の呼び出し時には、AWSのインフラ都合による瞬断などが発生しうるため、適切なエラー処理を入れておく。
  * [TimeoutSeconds](https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/sfn-stuck-execution.html) を指定して、 *Lambda* の呼び出しがスタックしないようにする。
  * [リトライ処理](docs.aws.amazon.com/ja_jp/step-functions/latest/dg/amazon-states-language-errors.html#amazon-states-language-retrying-after-error) を入れて、自動的に復旧させる。
    * どのようなエラーが発生するか、網羅的な情報がないので全エラー（`States.ALL`）を対象にリトライを入れておく。
    * エラーには継承関係がある。Java SDKの [APIドキュメント](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/lambda/model/AWSLambdaException.html) の継承関係を参照。

* *Lambda* の呼び出し時に `Lambda.ResourceNotFoundException` を無視することを検討する。
  * 例えば、複数のチームでステートマシンを共用しているような場合、他チームのLambda関数がなくてもステートマシンを実行したい場合など。
  * `Lambda.ResourceNotFoundException` が発生したときにリトライされるのを回避するため、`MaxAttempts: 0` を指定しておく。

```yaml
---
States:
  example:
    Type: Task
    Resource: arn:aws:lambda:us-east-1:9999999999:function:example:${opt:stage}
    TimeoutSeconds: 300
    End: true
    Retry:
      - ErrorEquals:
        - Lambda.ResourceNotFoundException
        MaxAttempts: 0
      - ErrorEquals:
        - States.All
        IntervalSeconds: 3
        MaxAttempts: 2
        BackoffRate: 1.5
    Catch:
      - ErrorEquals:
        - Lambda.ResourceNotFoundException
      Next: RNF-example
  RNF-example:
    Type: Pass
    ResultPath: "$.cause"
    End: true
```

