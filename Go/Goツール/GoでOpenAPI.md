# OpenAPIからGo言語のコードを自動生成
>[OpenAPIでコードを生成してスキーマ駆動で開発する：GoとTypeScriptの場合](https://note.com/navitime_tech/n/nd734b4fdfc1e#115ab377-d0ef-45a5-a44b-818a8784fc38)


# インストール
```bash
go install github.com/deepmap/oapi-codegen/cmd/oapi-codegen@latest
```
# OpenAPI定義
```yaml
openapi: "3.0.0"
info:
  version: 1.0.0
  title: Swagger Petstore
  license:
    name: MIT
servers:
  - url: http://petstore.swagger.io/v1
paths:
  /pets:
    get:
      summary: List all pets
      operationId: listPets
      tags:
        - pets
      parameters:
        - name: limit
          in: query
          description: How many items to return at one time (max 100)
          required: false
          schema:
            type: integer
            format: int32
      responses:
        '200':
          description: A paged array of pets
          headers:
            x-next:
              description: A link to the next page of responses
              schema:
                type: string
          content:
            application/json:    
              schema:
                $ref: "#/components/schemas/Pets"
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
    post:
      summary: Create a pet
      operationId: createPets
      tags:
        - pets
      responses:
        '201':
          description: Null response
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
  /pets/{petId}:
    get:
      summary: Info for a specific pet
      operationId: showPetById
      tags:
        - pets
      parameters:
        - name: petId
          in: path
          required: true
          description: The id of the pet to retrieve
          schema:
            type: string
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Pet"
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
components:
  schemas:
    Pet:
      type: object
      required:
        - id
        - name
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        tag:
          type: string
    Pets:
      type: array
      items:
        $ref: "#/components/schemas/Pet"
    Error:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: integer
          format: int32
        message:
          type: string
```

# 使い方
```bash
oapi-codegen -package "openapi" petstore.yaml > petstore.gen.go
```
すると以下のような内容が書かれた petstore.gen.go が生成されます。
- components や parameters のstruct定義
- 全APIのハンドラーを持った ServerInterface
- Echo用のwrapper
- Base64エンコードされたOpenAPI spec

生成対象はconfigファイルで設定することができます。
今回はstruct定義のみ生成したかったので、以下のような設定にしました。

><コード生成設定>.yaml
```yaml
package: openapi
generate:
  models: true
  # echo-server: true
  # embedded-spec: true
```
```bash
oapi-codegen -config ＜コード生成設定＞.yaml <OpenAPI定義>.yaml > ＜生成ファイル名＞.gen.go
```
- 生成ファイルはパスを指定することは出来ないから出来たらそれを適当なフォルダに入れてインポートする
## AWS Lambdaのハンドラー内で利用する
こちらのようにしてリクエストパラメータを生成コードにマッピングして、処理を行い、レスポンスを返すよう実装しました。
```go
package main

import (
	"encoding/json"
	"net/http"

	"github.com/aws/aws-lambda-go/events"
	"github.com/aws/aws-lambda-go/lambda"
	"github.com/mitchellh/mapstructure"

	"example.com/petstore/openapi"
)

func handler(request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
	// requestをopenapi.ListPetsParamsにマッピングする
	var param openapi.ListPetsParams

	decoderConfig := &mapstructure.DecoderConfig{
		WeaklyTypedInput: true,
		Result:           &param,
	}
	decoder, err := mapstructure.NewDecoder(decoderConfig)
	if err != nil {
		return events.APIGatewayProxyResponse{}, err
	}
	err = decoder.Decode(request.QueryStringParameters)
	if err != nil {
		return events.APIGatewayProxyResponse{}, err
	}

	// do something

	// openapi.Petsを作成してJSONにして返却する
	pets := make(openapi.Pets, 0)

	body, err := json.Marshal(pets)
	return events.APIGatewayProxyResponse{
		StatusCode: http.StatusOK,
		Body:       string(body),
	}, err
}

func main() {
	lambda.Start(handler)
}
```
こうすることで、生成されたコードとリクエスト、レスポンスのマッピングが行われ、仕様書とコードが一致するようになりました。

## フロントエンドのコード生成(TypeScript)
続きは[こちら](https://note.com/navitime_tech/n/nd734b4fdfc1e#32639449-e92d-4770-be21-ef098b2157ef)


























