API Gateway
====

# Lambdaプロキシ統合

* Lambda関数の入力は、定形のJSONになる。
  * [プロキシ統合のための Lambda 関数の入力形式](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format)
  * カスタム認証をかけている場合は、認証情報が `requestContext/authorizer/principalId` に入ってくる。
  * マルチパート形式のリクエストも扱える。入力JSONの `body` にマルチパート文字列がそのまま入ってくるので、パースするなりして処理する。

* Lambda関数の出力は、定形のJSONでなければならない。
  * [プロキシ統合のための Lambda 関数の出力形式](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-output-format)

* APIのタイムアウト時間は最大 29秒 までしか伸ばせない。
  * *Lambda* のタイムアウトをそれ以上伸ばしても意味がない。（*API G/W*はタイムアウトを返すが、＊Lambda*はそのまま動き続ける）

* CORSを有効にした場合は、Lambda関数からのレスポンスで `Access-Control-Allow-Origin` ヘッダを返す必要がある。
  * [Amazon API GatewayをCross-Originで利用する設定、Cross-Originせずに利用する設定のまとめ](https://qiita.com/aiwas/items/116a1039558bec1c5edd)
  * [AWS Lambda Proxy Integrationを試してみた](https://qiita.com/seiya_orz/items/2bd83204e212e35b2c6c#cors%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%E8%BF%BD%E8%A8%9820161116)

# カスタムドメイン

* この機能を使うことで、 *CloudFront* に API とマッピングされたディストリビューションが作成される。
  * API呼び出し時のパスと、呼び出す API（およびそのステージ）を柔軟にマッピングできるのが便利。逆にこの機能でなければ、 *API G/W* のステージにドメインをマッピングすることはできない。（Ex. `dev.api.example.com` -> `/dev`）

* 指定する *カスタムドメイン名* と重複する（包含する）ドメインが *CloudFront* に存在すると作成できない。
  * 例えば、すでに `*.example.com` と言うドメインが存在する場合は、`foo.example.com` や `bar.example.com` と言うカスタムドメインは作成できない。
  * SPAアプリを `CloudFront` で配信する場合、SPA と API は別ドメインにしておいたほうが良い。（Ex. API `*.api.example.com`、SPA `*.console.example.com`）

* 指定する *ACM証明書* は、`us-east-1` に作成しなければならない。（エッジ最適化の関係か？）

* 指定する *カスタムドメイン名* と、 *ACM証明書* のドメインは一致していないといけない。（カスタムドメインが作成されたかのように見えるが、 *CloudFront* にディストリビューションが作成されない）
