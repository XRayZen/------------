# 制約
GORMはタグを使用したデータベース制約の作成を可能にします。
- 指定した制約は、 GORMによるオートマイグレーション時、テーブル作成時 に作成されます

## CHECK制約
CHECK制約の作成はcheckタグで行います。
```go
type UserIndex struct {
    Name  string `gorm:"check:name_checker,name <> 'jinzhu'"`
    Name2 string `gorm:"check:name <> 'jinzhu'"`
    Name3 string `gorm:"check:,name <> 'jinzhu'"`
}
```
## インデックス制約
GORMインデックスを参照

## 外部キー制約
GORMはアソシエーションのための外部キー制約を作成します。初期化時にこの機能を無効にすることができます：
```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  DisableForeignKeyConstraintWhenMigrating: true,
})
```
GORMでは、外部キー制約のOnDeleteとOnUpdateオプションをconstraintタグを用いて設定することができます。
```go
type User struct {
  gorm.Model
  CompanyID  int
  Company    Company    `gorm:"constraint:OnUpdate:CASCADE,OnDelete:SET NULL;"`
  CreditCard CreditCard `gorm:"constraint:OnUpdate:CASCADE,OnDelete:SET NULL;"`
}

type CreditCard struct {
  gorm.Model
  Number string
  UserID uint
}

type Company struct {
  ID   int
  Name string
}
```
# 複合主キー
複数のフィールドを主キーとして設定すると、次のように複合主キーが作成されます。
```go
type Product struct {
  ID           string `gorm:"primaryKey"`
  LanguageCode string `gorm:"primaryKey"`
  Code         string
  Name         string
}
```
!!! attention intのPrioritizedPrimaryFieldはデフォルトでAutoIncrementされます。
    無効にしたいなら、autoIncrementをintフィールドでfalseにしてください。
```go
type Product struct {
  CategoryID uint64 `gorm:"primaryKey;autoIncrement:false"`
  TypeID     uint64 `gorm:"primaryKey;autoIncrement:false"`
}
```

