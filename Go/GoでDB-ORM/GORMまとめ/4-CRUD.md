- [データ操作（CRUD）](#データ操作crud)
  - [INSERT](#insert)
  - [SELECT](#select)
    - [様々な取得方法](#様々な取得方法)
      - [（取得方法①）全てのレコードを取得](#取得方法全てのレコードを取得)
      - [（取得方法②）単一値をカラム指定で取得(Select)](#取得方法単一値をカラム指定で取得select)
      - [(取得方法③)複数値をカラム指定で取得(Pluck)](#取得方法複数値をカラム指定で取得pluck)
      - [（取得方法④）最初のレコードを取得(ソート)](#取得方法最初のレコードを取得ソート)
      - [（取得方法⑤）最後のレコードを取得（ソート）](#取得方法最後のレコードを取得ソート)
      - [（取得方法⑥）最初のレコードを取得(ソートなし)](#取得方法最初のレコードを取得ソートなし)
      - [(取得方法⑦)プリロード](#取得方法プリロード)
    - [WHERE](#where)
    - [Not](#not)
    - [Or](#or)
    - [ソート](#ソート)
    - [Limit](#limit)
    - [Offset](#offset)
    - [Count](#count)
    - [Group](#group)
    - [Having](#having)
    - [FirstOrInit](#firstorinit)
    - [FirstOrCreate](#firstorcreate)
  - [UPDATE](#update)
    - [全フィールドの更新](#全フィールドの更新)
    - [特定のカラムのみを更新](#特定のカラムのみを更新)
  - [DELETE](#delete)
    - [論理削除](#論理削除)
    - [物理削除](#物理削除)
    - [削除失敗](#削除失敗)
  - [素のSQLを実行する](#素のsqlを実行する)

# データ操作（CRUD）
## INSERT
Create関数でデータを挿入します。

```go
var animal = Animal{Age: 99, Name: ""}
db.Create(&animal)
// INSERT INTO animals("age") values('99');
```

## SELECT
### 様々な取得方法
取得方法が複数用意されています。
取得した情報は引数で与えたモデルに格納されます。

#### （取得方法①）全てのレコードを取得
Find関数で全レコードを取得します。

```go
db.Find(&users)
//// SELECT * FROM users;
```

#### （取得方法②）単一値をカラム指定で取得(Select)
Select関数でカラムを指定します。
Table関数でテーブルを指定することもできます。

```go
db.Select("name, age").Find(&user)
//// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&user)
//// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
//// SELECT COALESCE(age,'42') FROM users;
```

#### (取得方法③)複数値をカラム指定で取得(Pluck)
Pluck関数で任意のカラムのスライスを取得します。

```go
var ages []int64
db.Model(&User{}).Pluck("age", &ages)
```

#### （取得方法④）最初のレコードを取得(ソート)
First関数で主キーでソートされた最初のレコードを一行取得します。
主キー以外でソートして一行取得したいのであれば、Order関数とTake関数を組み合わせる。

```go
db.First(&user)
//// SELECT * FROM users ORDER BY id LIMIT 1;
```

#### （取得方法⑤）最後のレコードを取得（ソート）
Last関数で主キーでソートされた最後のレコードを一行取得します。

```go
db.Last(&user)
//// SELECT * FROM users ORDER BY id DESC LIMIT 1;
```

#### （取得方法⑥）最初のレコードを取得(ソートなし)
Take関数でソートなしで最初のレコードを一行取得します。

```go
db.Take(&user)
//// SELECT * FROM users LIMIT 1;
```

#### (取得方法⑦)プリロード
プリロードを利用すれば、1センテンスで複数のテーブルからデータを取得できます。
モデル間の関係を持っていることが前提になります。
- 関係を持っていると自動でプリロード対象になるため、プリロードしたくない場合は、`gorm:"PRELOAD:false"`をタグします。

```go
type User struct {
  gorm.Model
  Name       string
  CompanyID  uint
  Company    Company `gorm:"PRELOAD:false"` // not preloaded
  Role       Role                           // preloaded
}
```

最も単純なプリロードの例は下記です。

```go
db.Preload("Orders").Find(&users)
//// SELECT * FROM users;
//// SELECT * FROM orders WHERE user_id IN (取得したusersのID);
```

プリロード対象に対して抽出条件を付ける場合は、下記のようになります。

```go
db.Preload("Orders", "state NOT IN (?)", "cancelled").Find(&users)
//// SELECT * FROM users;
//// SELECT * FROM orders WHERE user_id IN (取得したusersのID) AND state NOT IN ('cancelled');
```

基底となるテーブルとプリロード対象の両方に対して抽出条件を付ける場合は、下記のようになります。

```go
db.Where("state = ?", "active").Preload("Orders", "state NOT IN (?)", "cancelled").Find(&users)
//// SELECT * FROM users WHERE state = 'active';
//// SELECT * FROM orders WHERE user_id IN (取得したusersのID) AND state NOT IN ('cancelled');
```

3つプリロードする場合は、下記のようになります。

```go
db.Preload("Orders").Preload("Profile").Preload("Role").Find(&users)
//// SELECT * FROM users;
//// SELECT * FROM orders WHERE user_id IN (取得したusersのID); // has many
//// SELECT * FROM profiles WHERE user_id IN (取得したusersのID); // has one
//// SELECT * FROM roles WHERE id IN (取得したusersのロールID); // belongs to
```

### WHERE
Where関数で条件を指定します。
プレースホルダを使えます。

```go
// 条件に一致した最初のレコードを取得します
db.Where("name = ?", "jinzhu").First(&user)
//// SELECT * FROM users WHERE name = 'jinzhu' limit 1;

// 条件に一致したすべてのレコードを取得します
db.Where("name = ?", "jinzhu").Find(&user)
//// SELECT * FROM users WHERE name = 'jinzhu';

// <>
db.Where("name <> ?", "jinzhu").Find(&user)

// IN
db.Where("name in (?)", []string{"jinzhu", "jinzhu 2"}).Find(&user)
db.Where([]int64{20, 21, 22}).Find(&users)
//// SELECT * FROM users WHERE id IN (20, 21, 22);

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&user)

// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&user)

// BETWEEN
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&user)
```

構造体やマップをWhereに指定すると、そのまま条件として扱われます。

```go
// Struct
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
//// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&user)
//// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;
```

### Not
```go
db.Not("name", "jinzhu").First(&user)
//// SELECT * FROM users WHERE name <> "jinzhu" LIMIT 1;
```

### Or
```go
db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
//// SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';
```

### ソート
Order関数でソートします。

```go
db.Order("age desc, name").Find(&users)
//// SELECT * FROM users ORDER BY age desc, name;

// Multiple orders
db.Order("age desc").Order("name").Find(&users)
//// SELECT * FROM users ORDER BY age desc, name;

// ReOrder
db.Order("age desc").Find(&users1).Order("age", true).Find(&users2)
//// SELECT * FROM users ORDER BY age desc; (users1)
//// SELECT * FROM users ORDER BY age; (users2)
```

### Limit
Limit関数で取得件数を指定します。

```go
db.Limit(3).Find(&users)
//// SELECT * FROM users LIMIT 3;

// Cancel limit condition with -1
db.Limit(10).Find(&users1).Limit(-1).Find(&users2)
//// SELECT * FROM users LIMIT 10; (users1)
//// SELECT * FROM users; (users2)
```

### Offset
Offset関数で取得レコードの先頭いくつをスキップするかを指定します。」

```go
db.Offset(3).Find(&users)
//// SELECT * FROM users OFFSET 3;

// Cancel offset condition with -1
db.Offset(10).Find(&users1).Offset(-1).Find(&users2)
//// SELECT * FROM users OFFSET 10; (users1)
//// SELECT * FROM users; (users2)
```

### Count
Count関数で取得レコード数を取得します。

```go
db.Where("name = ?", "jinzhu").Or("name = ?", "jinzhu 2").Find(&users).Count(&count)
//// SELECT * from USERS WHERE name = 'jinzhu' OR name = 'jinzhu 2'; (users)
//// SELECT count(*) FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2'; (count)

db.Model(&User{}).Where("name = ?", "jinzhu").Count(&count)
//// SELECT count(*) FROM users WHERE name = 'jinzhu'; (count)

db.Table("deleted_users").Count(&count)
//// SELECT count(*) FROM deleted_users;
```

### Group
Group関数で指定カラムでのグループ化します。

```go
rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Rows()
for rows.Next() {
    ...
}
```

### Having
Having関数でグループ化したものを条件判定します。

```go
rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Rows()
for rows.Next() {
    ...
}

type Result struct {
    Date  time.Time
    Total int64
}
db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Scan(&results)
```

### FirstOrInit
FirstOrInit関数で、指定した条件でレコードが存在していた場合は最初のレコードを取得しモデルを初期化します。存在しなければその条件で*モデルを初期化*します。

```go
// Found
db.Where(User{Name: "Jinzhu"}).FirstOrInit(&users)
//// user -> User{Id: 111, Name: "Jinzhu", Age: 20}
db.FirstOrInit(&user, map[string]interface{}{"name": "jinzhu"})
//// user -> User{Id: 111, Name: "Jinzhu", Age: 20}

// Unfound
db.FirstOrInit(&user, User{Name: "non_existing"})
//// user -> User{Name: "non_existing"}
```

存在しなかった場合に、追加する情報を増やす場合はAttrs関数を併用します。

```go
// Unfound
db.Where(User{Name: "non_existing"}).Attrs(User{Age: 20}).FirstOrInit(&users)
//// SELECT * FROM USERS WHERE name = 'non_existing';
//// user -> User{Name: "non_existing", Age: 20}

db.Where(User{Name: "non_existing"}).Attrs("age", 20).FirstOrInit(&users)
//// SELECT * FROM USERS WHERE name = 'non_existing';
//// user -> User{Name: "non_existing", Age: 20}

// Found
db.Where(User{Name: "Jinzhu"}).Attrs(User{Age: 30}).FirstOrInit(&users)
//// SELECT * FROM USERS WHERE name = jinzhu';
//// user -> User{Id: 111, Name: "Jinzhu", Age: 20}
```

また、存在するしないに関わらずモデルを設定する場合はAssign関数を使用します。

```go
// Unfound
db.Where(User{Name: "non_existing"}).Assign(User{Age: 20}).FirstOrInit(&users)
//// user -> User{Name: "non_existing", Age: 20}

// Found
db.Where(User{Name: "Jinzhu"}).Assign(User{Age: 30}).FirstOrInit(&users)
//// SELECT * FROM USERS WHERE name = jinzhu';
//// user -> User{Id: 111, Name: "Jinzhu", Age: 30}
```

### FirstOrCreate
FirstOrCreate関数で、指定した条件でレコードが存在していた場合は最初のレコードを取得しモデルを初期化します。存在しなければその条件で*レコードを保存*します。

```go
// Found
db.Where(User{Name: "Jinzhu"}).FirstOrCreate(&users)
//// user -> User{Id: 111, Name: "Jinzhu"}

// Unfound
db.FirstOrCreate(&users, User{Name: "non_existing"})
//// INSERT INTO "users" (name) VALUES ("non_existing");
//// user -> User{Id: 112, Name: "non_existing"}
```

存在しなかった場合に、追加する情報を増やす場合はAttrs関数を併用します。

```go
// Unfound
db.Where(User{Name: "non_existing"}).Attrs(User{Age: 20}).FirstOrCreate(&users)
//// SELECT * FROM users WHERE name = 'non_existing';
//// INSERT INTO "users" (name, age) VALUES ("non_existing", 20);
//// user -> User{Id: 112, Name: "non_existing", Age: 20}

// Found
db.Where(User{Name: "jinzhu"}).Attrs(User{Age: 30}).FirstOrCreate(&users)
//// SELECT * FROM users WHERE name = 'jinzhu';
//// user -> User{Id: 111, Name: "jinzhu", Age: 20}
```

また、存在するしないに関わらずレコードを挿入あるいは更新する場合はAssign関数を使用します。
いわゆるpostgresqlのUPSERTを処理したい場合はこれが該当します。

```go
// Unfound
db.Where(User{Name: "non_existing"}).Assign(User{Age: 20}).FirstOrCreate(&users)
//// SELECT * FROM users WHERE name = 'non_existing';
//// INSERT INTO "users" (name, age) VALUES ("non_existing", 20);
//// user -> User{Id: 112, Name: "non_existing", Age: 20}

// Found
db.Where(User{Name: "jinzhu"}).Assign(User{Age: 30}).FirstOrCreate(&users)
//// SELECT * FROM users WHERE name = 'jinzhu';
//// UPDATE users SET age=30 WHERE id = 111;
//// user -> User{Id: 111, Name: "jinzhu", Age: 30}
```
## UPDATE
### 全フィールドの更新
Save関数で全フィールドを更新します。

```go
db.First(&user)
user.Name = "jinzhu 2"
user.Age = 100

db.Save(&user)
//// UPDATE users SET name='jinzhu 2', age=100, birthday='2016-01-01', updated_at = '2013-11-17 21:34:10' WHERE id=111;
```

ただし、構造体のフィールドにゼロ値が含まれているとUpdate処理は実行されないようですのでご注意ください。

http://gorm.io/docs/update.html#Extra-Updating-option

> Update with struct only works with none zero values,

### 特定のカラムのみを更新
UpdateあるいはUpdates関数で特定のカラムのみを更新します。

```go
// nameカラムの値を"hello"に更新します
db.Model(&user).Update("name", "hello")
//// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// 条件付き(active==trueならば)でnameカラムの値を"hello"に更新します
db.Model(&user).Where("active = ?", true).Update("name", "hello")
//// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;

// `map` で複数のフィールドを更新します(対象のフィールドのみ)
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "actived": false})
//// UPDATE users SET name='hello', age=18, actived=false, updated_at='2013-11-17 21:34:10' WHERE id=111;

// `struct` で複数のフィールドを更新します(空ではないフィールドのみ)
db.Model(&user).Updates(User{Name: "hello", Age: 18})
//// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;
```

## DELETE
### 論理削除
テーブルに`DeletedAt`フィールドが存在する場合に、`Delete`関数を実行すると自動で論理削除になります。

```go
db.Delete(&user)
//// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE id = 111;
```

### 物理削除
テーブルに`DeletedAt`フィールドが存在する場合に物理削除したい場合は、Unscoped().Delete関数を利用します。

```go
db.Unscoped().Delete(&order)
//// DELETE FROM orders WHERE id=10;
```

また、テーブルに`DeletedAt`フィールドが存在しない場合に、`Delete`関数を実行すると物理削除になります。

```go
db.Delete(&email)
//// DELETE from emails where id=10;
```

### 削除失敗
たとえ、条件にあうレコードが見つからずに、削除に失敗したとしてもエラーにはなりません。
削除失敗したときにエラーとしたい場合は、 `RowsAffected` を活用します。
サンプルコードとissueはこちらです。
https://github.com/jinzhu/gorm/issues/1380#issuecomment-600342388

## 素のSQLを実行する
SELECTはRaw関数、その他はExec関数で素のSQLを引数に渡して実行します。

```go
type Result struct {
    Name string
    Age  int
}
var result Result
db.Raw("SELECT name, age FROM users WHERE name = ?", 3).Scan(&result)

db.Exec("DROP TABLE users;")
db.Exec("UPDATE orders SET shipped_at=? WHERE id IN (?)", time.Now(), []int64{11,22,33})
```

素のSQLも実行結果は`*sql.Row`や`*sql.Rows`として取得できます。

```go
rows, err := db.Raw("select name, age, email from users where name = ?", "jinzhu").Rows() // (*sql.Rows, error)
defer rows.Close()
for rows.Next() {
    ...
    rows.Scan(&name, &age, &email)
    ...
}
```