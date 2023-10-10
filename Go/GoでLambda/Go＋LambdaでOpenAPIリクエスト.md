# OpenAPIのリクエストをパース
- 判明しているのが2種類ある
- 違いはAPI定義でパラメータにしているか、ボディにしているかで違う
# Headerで変換
>[OpenAPIでコードを生成してスキーマ駆動で開発する：GoとTypeScriptの場合](https://note.com/navitime_tech/n/nd734b4fdfc1e)
## OpenAPIの定義
```yaml
 /pets:
    get:
      parameters:
        - name: limit
          in: query
          description: How many items to return at one time (max 100)
          required: false
          schema:
            type: integer
            format: int32
        - name: sort-by
          in: query
          description: How to sort items
          required: false
          schema:
            type: string
```

### AWS Lambdaのハンドラー内で利用する
ハンドラーのコードについて詳細は省きますが、
こちらのようにしてリクエストパラメータを生成コードにマッピングして、処理を行い、レスポンスを返すよう実装しました。
```go
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
```
こうすることで、生成されたコードとリクエスト、レスポンスのマッピングが行われ、仕様書とコードが一致するようになりました。
## 実際のコード例
```go
// リクエストをヘッダーからパース
func parseHeaderRequest(request events.APIGatewayProxyRequest) (api_gen_code.PostUserJSONBody, error) {
	var api_req api_gen_code.PostUserJSONBody
	decoder_config := &mapstructure.DecoderConfig{
		WeaklyTypedInput: true,
		Result:           &api_req,
	}
	decoder, err := mapstructure.NewDecoder(decoder_config)
	if err != nil {
		return api_gen_code.PostUserJSONBody{}, err
	}
	err = decoder.Decode(request.QueryStringParameters)
	if err != nil {
		return api_gen_code.PostUserJSONBody{}, err
	}
	return api_req, nil
}
```
# request.Bodyをjson.Marshalで変換
>[AWS SAM + OpenAPI(Swagger) + Golang でAPIを構築する](https://tech.buysell-technologies.com/entry/2021/06/10/104135)
## OpenAPIの定義
```yaml
    post:
      summary: POSTサンプル
      description: POSTのサンプルです
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  description: 名前
                  type: string
                age:
                  description: 年齢
                  type: integer
                  format: int32
              required:
                - name
                - age
```
## Go
```go
func handler(request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {

    var p Person
    b := []byte(request.Body)

    err := json.Unmarshal(b, &p)
    if err != nil {
        panic(err)
    }

    body := Response{Message: fmt.Sprintf("Hello %v : %v", p.Name, p.Age)}
    jsonBody, err := json.Marshal(body)
    if err != nil {
        panic(err)
    }

    return events.APIGatewayProxyResponse{
        Body:       string(jsonBody),
        StatusCode: 200,
    }, nil
}
```
## 実際のコード例
```go
// リクエストをボディからパース
func parseBodyRequest(request events.APIGatewayProxyRequest) (api_gen_code.PostUserJSONBody, error) {
	var api_req api_gen_code.PostUserJSONBody
	if err := json.Unmarshal([]byte(request.Body), &api_req); err != nil {
		return api_gen_code.PostUserJSONBody{}, err
	}
	return api_req, nil
}
```
