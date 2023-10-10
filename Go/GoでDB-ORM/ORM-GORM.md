# GORM
>[公式サイト](https://gorm.io/docs/index.html)

The fantastic ORM library for Golang aims to be developer friendly.

## Overview
- フル装備のORM
- アソシエーション（Has One, Has Many, Belongs To, Many To Many, Polymorphism, Single-table inheritance)
- フック（作成/保存/更新/削除/検索の前/後）
- Preloadによるイージーロード、Joins
- トランザクション、ネストされたトランザクション、セーブポイント、セーブポイントへのRollbackTo
- コンテキスト、プリペアド・ステートメント・モード、ドライラン・モード
- バッチ挿入、FindInBatches、Mapによる検索/作成、SQL ExprとContext ValuerによるCRUD
- SQL Builder、Upsert、Locking、Optimizer/Index/Comment Hints、Named Argument、SubQuery
- 複合主キー、インデックス、制約条件
- 自動移行
- ロガー
- 拡張可能で柔軟なプラグインAPIデータベースリゾルバ（複数データベース、読み書き分割）/ Prometheus...
- すべての機能にはテストが含まれています
- 開発者フレンドリー

## Install
```bash
go get -u gorm.io/gorm
go get -u gorm.io/driver/sqlite
```
## Quick Start
```go
package main

import (
  "gorm.io/gorm"
  "gorm.io/driver/sqlite"
)

type Product struct {
  gorm.Model
  Code  string
  Price uint
}

func main() {
  db, err := gorm.Open(sqlite.Open("test.db"), &gorm.Config{})
  if err != nil {
    panic("failed to connect database")
  }

  // Migrate the schema
  db.AutoMigrate(&Product{})

  // Create
  db.Create(&Product{Code: "D42", Price: 100})

  // Read
  var product Product
  db.First(&product, 1) // find product with integer primary key
  db.First(&product, "code = ?", "D42") // find product with code D42

  // Update - update product's price to 200
  db.Model(&product).Update("Price", 200)
  // Update - update multiple fields
  db.Model(&product).Updates(Product{Price: 200, Code: "F42"}) // non-zero fields
  db.Model(&product).Updates(map[string]interface{}{"Price": 200, "Code": "F42"})

  // Delete - delete product
  db.Delete(&product, 1)
}
```
# GORMのAutoMigrateの意味
`AutoMigrate`は、GORMの機能の1つで、データベースのスキーマを自動的にマイグレーションするために使用されます。つまり、GORMのモデルを定義し、それらをデータベースにマッピングすると、`AutoMigrate`を使用して、データベースのテーブルを自動的に作成または更新できます。

`AutoMigrate`は、GORMのモデルとデータベースのテーブルの間のマッピングを自動的に行います。つまり、GORMのモデルに新しいフィールドを追加した場合、`AutoMigrate`を使用してデータベースのテーブルを更新できます。また、データベースのテーブルに新しいカラムを追加した場合、`AutoMigrate`を使用してGORMのモデルを更新できます。

以下は、`AutoMigrate`の使用例です。

```go
import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/mysql"
)

type User struct {
    gorm.Model
    Name string
    Age  int
}

func main() {
    // RDSデータベースに接続する
    db, err := RDS_Connect("mysql", "mydb", "myuser", "mypassword", "mydbinstance.123456789012.us-east-1.rds.amazonaws.com:3306")
    if err != nil {
        panic(err)
    }
    defer db.Close()

    // テーブルを自動的に作成または更新する
    db.AutoMigrate(&User{})

    // ...
}
```

このように、`AutoMigrate`を使用して、GORMのモデルとデータベースのテーブルを自動的にマッピングできます。


