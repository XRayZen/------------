# インデックス
GORMでは index や uniqueIndex タグを使用して、インデックスを作成することができます。
- 定義されたインデックスは GORMの AutoMigrate や CreateTable 実行時に作成されます。

## indexタグ
class, type, where, comment, expression, sort, collate, option など、多くのindexの設定を指定できます。

使用方法については、以下の例を参照してください。

```go
type User struct {
    Name  string `gorm:"index"`
    Name2 string `gorm:"index:idx_name,unique"`
    Name3 string `gorm:"index:,sort:desc,collate:utf8,type:btree,length:10,where:name3 != 'jinzhu'"`
    Name4 string `gorm:"uniqueIndex"`
    Age   int64  `gorm:"index:,class:FULLTEXT,comment:hello \\, world,where:age > 10"`
    Age2  int64  `gorm:"index:,expression:ABS(age)"`
}

// MySQL option
type User struct {
    Name string `gorm:"index:,class:FULLTEXT,option:WITH PARSER ngram INVISIBLE"`
}

// PostgreSQL option
type User struct {
    Name string `gorm:"index:,option:CONCURRENTLY"`
}
```
## uniqueIndex
uniqueIndex タグは index と似た働きを持ち、 index:,unique を指定するのと等しくなります。
```go
type User struct {
    Name1 string `gorm:"uniqueIndex"`
    Name2 string `gorm:"uniqueIndex:idx_name,sort:desc"`
}
```
## 複合インデックス
2つのフィールドに同じインデックス名を使用すると、複合インデックスが作成されます。

```go
// create composite index `idx_member` with columns `name`, `number`
type User struct {
    Name   string `gorm:"index:idx_member"`
    Number string `gorm:"index:idx_member"`
}
```
## フィールドの優先度
複合インデックスを指定する際、カラムの順序がパフォーマンスに影響を与えます。そのため、順序は慎重に指定する必要があります。

priority オプションで順序を指定できます。デフォルトの優先度の値は 10です。 priorityの値が同じ場合、順序は構造体のフィールドの並びに基づいて設定されます。
```go
type User struct {
    Name   string `gorm:"index:idx_member"`
    Number string `gorm:"index:idx_member"`
}
// column order: name, number

type User struct {
    Name   string `gorm:"index:idx_member,priority:2"`
    Number string `gorm:"index:idx_member,priority:1"`
}
// column order: number, name

type User struct {
    Name   string `gorm:"index:idx_member,priority:12"`
    Number string `gorm:"index:idx_member"`
}
// column order: number, name
```
# 共有複合インデックス
埋め込み構造体で共有複合インデックスを作成する場合、構造体を複数回埋め込むとdbのインデックス名が重複してしまうため、インデックス名を指定することはできません。

この場合、コンポジットインデックスのIDを意味するcompositeというインデックスタグを使用できます。元のルールと同じように、構造体のコンポジット ID が同じフィールドはすべて同じインデックスにまとめられます。しかし、改良された点は、最も派生/埋め込み構造体がNamingStrategyによってインデックスの名前を生成できるようになったことです。

たとえば
```go
type Foo struct {
  IndexA int `gorm:"index:,unique,composite:myname"`
  IndexB int `gorm:"index:,unique,composite:myname"`
}
```
Fooテーブルが作成された場合、複合インデックスの名前はidx_foo_mynameとなります。
```go
type Bar0 struct {
  Foo
}

type Bar1 struct {
  Foo
}
```



















