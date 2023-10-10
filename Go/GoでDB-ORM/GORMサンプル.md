- [RDSProxyとORMを組み合わせる](#rdsproxyとormを組み合わせる)
- [AWS RDS+Golang+GORMなテストリポジトリ](#aws-rdsgolanggormなテストリポジトリ)
  - [接続](#接続)
  - [テーブルを作成・移行](#テーブルを作成移行)


# RDSProxyとORMを組み合わせる
>https://qiita.com/maika_kamada/items/6eb6a40c17b4b8995acb#8-lambda%E9%96%A2%E6%95%B0%E3%81%AE%E4%BD%9C%E6%88%90

```go
package main

import (
    "database/sql"
    "encoding/json"
    "fmt"
    "github.com/aws/aws-lambda-go/lambda"
    _ "github.com/go-sql-driver/mysql"
    "os"
)

type Response struct {
    ID       int            `json:"id"`
    Name     string         `json:"name"`
    Champion string         `json:"champion"`
    Formed   string         `json:"formed"`
    Note     sql.NullString `json:"note"`
}

// os.Getenv（）でLambdaの環境変数を取得
var dbUser = os.Getenv("dbUser")         // DBに作成したユーザ名
var dbPass = os.Getenv("dbPass")         // パスワード
var dbEndpoint = os.Getenv("dbEndpoint") // RDS Proxyのプロキシエンドポイント
var dbName = os.Getenv("dbName")         // テーブルを作ったDB名

func RDSConnect() (*sql.DB, error) {
    connectStr := fmt.Sprintf(
        "%s:%s@tcp(%s:%s)/%s?charset=%s",
        dbUser,
        dbPass,
        dbEndpoint,
        "3306",
        dbName,
        "utf8",
    )
    db, err := sql.Open("mysql", connectStr)
    if err != nil {
        panic(err.Error())
    }
    return db, nil
}

func RDSProcessing(db *sql.DB) (interface{}, error) {

    var id int
    var name string
    var champion string
    var formed string
    var note sql.NullString

    responses := []Response{}
    responseMap := Response{}

    getData, err := db.Query("SELECT * FROM m1_champion")
    defer getData.Close()
    if err != nil {
        return nil, err
    }

    for getData.Next() {
        if err := getData.Scan(&id, &name, &champion, &formed, &note); err != nil {
            return nil, err
        }
        fmt.Println(id, name, champion, formed, note)
        responseMap.ID = id
        responseMap.Name = name
        responseMap.Champion = champion
        responseMap.Formed = formed
        responseMap.Note = note
        responses = append(responses, responseMap)
    }

    params, _ := json.Marshal(responses)
    fmt.Println(string(params))

    defer db.Close()
    return string(params), nil
}

func run() (interface{}, error) {
    fmt.Println("RDS接続 start!")
    db, err := RDSConnect()
    if err != nil {
        panic(err.Error())
    }
    fmt.Println("RDS接続 end!")
    fmt.Println("RDS処理 start!")
    response, err := RDSProcessing(db)
    if err != nil {
        panic(err.Error())
    }
    fmt.Println("RDS処理 end!")
    return response, nil
}

/**************************
   メイン
**************************/
func main() {
    lambda.Start(run)
}
```
# AWS RDS+Golang+GORMなテストリポジトリ
Amazon Linux2 + AWS RDS(mysql)でgormを使って簡単なCRUDなAPIを実装するテスト
ユーザー名、パスワードを扱うシンプルなAPIです。
>https://github.com/yasutakatou/goRDStest/tree/master
>[ソースコード](https://github.com/yasutakatou/goRDStest/blob/master/api.go#L458)

## 接続
```go
func GormConnect() *gorm.DB {
	DBTYPE := "mysql"
	CONNECT := dbUser + ":" + dbPass + "@tcp(" + dbAddress + ":3306)/test?charset=utf8&parseTime=True&loc=Local"

	DB, err := gorm.Open(DBTYPE, CONNECT)
	if err != nil {
		fmt.Println("RDS access error!")
		panic(err.Error())
	}

	DB.LogMode(true)
	DB.SingularTable(true)
	DB.AutoMigrate(&member{})

	return DB
}
```
## テーブルを作成・移行
```go
func GormMigrateTable(DBMS *gorm.DB) *gorm.DB {
	DBMS.AutoMigrate(&TestTable{})
	return DBMS
}
```





























