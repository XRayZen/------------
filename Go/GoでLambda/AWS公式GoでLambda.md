# Go による Lambda 関数の構築
>[公式](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/lambda-golang.html)

!!! info Go 1.x などの Amazon Linux オペレーティングシステムを使用するランタイムは、arm64 アーキテクチャをサポートしません。arm64 アーキテクチャを使用するには、provided.al2 ランタイムで Go を実行します。

## AWS公式サンプル
[Github公式リポジトリ](https://github.com/awsdocs/aws-lambda-developer-guide/tree/main/sample-apps/blank-go/function)

## Go 用のツールとライブラ
- [AWS SDK for Go](https://github.com/aws/aws-sdk-go)
  - Go プログラミング言語用の公式の AWS SDK。
- `github.com/aws/aws-lambda-go/lambda`
  - Go 用の Lambda プログラミングモデルの実装
  - このパッケージは、ハンドラーを呼び出すために AWS Lambda で使用されます。
- `github.com/aws/aws-lambda-go/lambdacontext`
  - コンテキストオブジェクトからコンテキスト情報にアクセスするためのヘルパー。
- `github.com/aws/aws-lambda-go/events`
  - このライブラリは一般的なイベントソース統合のタイプの定義を提供します。

詳細については、GitHub の「[aws-lambda-go](https://github.com/aws/aws-lambda-go)」をご参照ください。
















































