
- [GoでLambdaプロジェクト作成手順](#goでlambdaプロジェクト作成手順)
  - [ドッカーファイルを作成](#ドッカーファイルを作成)
- [Goで環境変数を取得](#goで環境変数を取得)
- [Goでシークレットマネージャのシークレットを取得](#goでシークレットマネージャのシークレットを取得)
  - [IaCでシークレットを定義](#iacでシークレットを定義)
  - [Goでシークレットマネージャのシークレットを取得](#goでシークレットマネージャのシークレットを取得-1)

# GoでLambdaプロジェクト作成手順
1. プロジェクト作成
    ```bash
    > mkdir <プロジェクト名>
    > cd <プロジェクト名>
    ```
2. モジュール宣言
    ```bash
    > go mod init <プロジェクト名>
    ```
1. AWS Lambdaパッケージをインストール
    ```bash
    > go get github.com/aws/aws-lambda-go/lambda
    ```
1. ハンドラー関数を作成
    ```go
    package main
    import (
        "context"
        "fmt"
        "github.com/aws/aws-lambda-go/lambda"
    )
    type MyEvent struct {
        Name string `json:"name"`
    }
    func HandleRequest(_ context.Context, name MyEvent) (string, error) {
        return fmt.Sprintf("Hello %s!", name.Name), nil
    }
    func main() {
        lambda.Start(HandleRequest)
    }
    ```
## ドッカーファイルを作成
!!! info Dockerfile で指定する Go のバージョン (たとえば、golang:1.20) が、アプリケーションの作成に使用した Go のバージョンと同じであることを確認してください。
```dockerfile
FROM golang:1.20 as build
WORKDIR /helloworld
# Copy dependencies list
COPY go.mod go.sum ./
# build
COPY hello.go .
RUN go build -o main hello.go
# copy artifacts to a clean image
FROM public.ecr.aws/lambda/provided:al2
COPY --from=build /helloworld/main /main
ENTRYPOINT [ "/main" ]
```
# Goで環境変数を取得
- 以下のIaCで環境変数を定義する
```bash
----infra/stage/develop/lambda_read/terragrunt.hcl----
---inputs---
variables = {
    rds_endpoint: dependency.rds.outputs.rds_proxy_endpoint,
    # RDSエンドポイント以外はEnv.hclから読み込むにした方が良い
    port : local.env.locals.db_port,
    usename : local.env.locals.db_username,
    db_name : local.env.locals.db_name,
    # パスワードはこれでいいのか？疑問
    db_pass : dependency.rds.outputs.db_password
}
-----resource "aws_lambda_function" -----
environment {
    variables = var.variables
}
```
- Goで環境変数を取得
```go
package main
import (
    "fmt"
    "os"
)

func main() {
    fmt.Println(os.Getenv("ENV"))
}
```
# Goでシークレットマネージャのシークレットを取得
## IaCでシークレットを定義
```bash
# RDSのテラグラント
inputs = {
        # DBユーザー名
    db_username = local.env.locals.db_username
    secret_version_stages = ["DEVELOP"]
}
# シークレットマネージャ
resource "random_password" "db-password" {
  length           = 16
  special          = true
  override_special = "_!%^"
}

resource "aws_secretsmanager_secret" "superuser" {
  name        = var.db_username
  description = "Database superuser, ${var.db_username}, database connection values"

  tags = var.tags
}

resource "aws_secretsmanager_secret_version" "superuser" {
  secret_id = aws_secretsmanager_secret.superuser.id
  version_stages = var.secret_version_stages
  secret_string = jsonencode({
    username = var.db_username
    password = random_password.db-password.result
  })
}
```


## Goでシークレットマネージャのシークレットを取得
```go
package main

import (
	"encoding/base64"
	"fmt"

	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/awserr"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/secretsmanager"
)

func main() {

	//名前とリージョンの定義
	secretName := "sample-secrets-name"
	region := "ap-northeast-1"

	//Secrets Manager clientを作成します
	svc := secretsmanager.New(session.New(),
		aws.NewConfig().WithRegion(region))
	input := &secretsmanager.GetSecretValueInput{
		SecretId:     aws.String(secretName),
		VersionStage: aws.String("AWSCURRENT"), // VersionStage デフォルトは、AWSCURRENTらしい。
	}

	result, err := svc.GetSecretValue(input)
	// たくさんエラーハンドリングされています。
	if err != nil {
		if aerr, ok := err.(awserr.Error); ok {
			switch aerr.Code() {
			case secretsmanager.ErrCodeDecryptionFailure:
				// Secrets Manager can't decrypt the protected secret text using the provided KMS key.
				fmt.Println(secretsmanager.ErrCodeDecryptionFailure, aerr.Error())

			case secretsmanager.ErrCodeInternalServiceError:
				// An error occurred on the server side.
				fmt.Println(secretsmanager.ErrCodeInternalServiceError, aerr.Error())

			case secretsmanager.ErrCodeInvalidParameterException:
				// You provided an invalid value for a parameter.
				fmt.Println(secretsmanager.ErrCodeInvalidParameterException, aerr.Error())

			case secretsmanager.ErrCodeInvalidRequestException:
				// You provided a parameter value that is not valid for the current state of the resource.
				fmt.Println(secretsmanager.ErrCodeInvalidRequestException, aerr.Error())

			case secretsmanager.ErrCodeResourceNotFoundException:
				// We can't find the resource that you asked for.
				fmt.Println(secretsmanager.ErrCodeResourceNotFoundException, aerr.Error())
			}
		} else {
			// Print the error, cast err to awserr.Error to get the Code and
			// Message from an error.
			fmt.Println(err.Error())
		}
		return
	}
	fmt.Println("result", result)

	//取得したsecretが文字列化バイナリかを判断します
	var secretString, decodedBinarySecret string
	if result.SecretString != nil {
		secretString = *result.SecretString
	} else {
		decodedBinarySecretBytes := make([]byte, base64.StdEncoding.DecodedLen(len(result.SecretBinary)))
		len, err := base64.StdEncoding.Decode(decodedBinarySecretBytes, result.SecretBinary)
		if err != nil {
			fmt.Println("Base64 Decode Error:", err)
			return
		}
		decodedBinarySecret = string(decodedBinarySecretBytes[:len])
	}

	fmt.Println("secretString:", secretString)
	fmt.Println("decodedBinarySecret", decodedBinarySecret)
}
```












































