# レコードの取得
>[GORM](https://gorm.io/ja_JP/docs/query.html)

- [レコードの取得](#レコードの取得)
  - [単一のオブジェクトを取得する](#単一のオブジェクトを取得する)
- [プライマリキーでオブジェクトを取得する](#プライマリキーでオブジェクトを取得する)
- [すべてのオブジェクトを取得する(全件取得)](#すべてのオブジェクトを取得する全件取得)
- [条件検索](#条件検索)
  - [文字列条件](#文字列条件)
  - [構造体 \& マップでの条件指定](#構造体--マップでの条件指定)
  - [構造体の検索フィールドを指定する](#構造体の検索フィールドを指定する)
  - [インライン条件](#インライン条件)
  - [Not条件](#not条件)
  - [特定のフィールドのみ選択](#特定のフィールドのみ選択)
  - [フィールドの選択](#フィールドの選択)
- [順序](#順序)
- [リミット＆オフセット](#リミットオフセット)
- [Group By \& Having](#group-by--having)
- [特徴的](#特徴的)
- [Joins（結合）](#joins結合)
- [プリロードに結合](#プリロードに結合)
  - [Join with conditions](#join-with-conditions)
  - [導出表の結合](#導出表の結合)
- [Scan](#scan)


## 単一のオブジェクトを取得する
単体取得には主に3つのメソッドがあります。

!!! info 特に条件が無ければ、Takeになる
- First
  - プライマリーキーの昇順で取得
  - プライマリーキーが無い場合は、モデルの最初のフィールドで順序付けされる。
- Take
  - 特に条件を指定せず取得
- Last
  - プライマリーキーの降順で取得
  - プライマリーキーが無い場合は、モデルの最初のフィールドで順序付けされる。

それらのメソッドは、データベースにクエリを実行する際にLIMIT 1の条件を追加し、レコードが見つからなかった場合、ErrRecordNotFoundエラーを返します。
```go
// Get the first record ordered by primary key
db.First(&user)
// SELECT * FROM users ORDER BY id LIMIT 1;

// Get one record, no specified order
db.Take(&user)
// SELECT * FROM users LIMIT 1;

// Get last record, ordered by primary key desc
db.Last(&user)
// SELECT * FROM users ORDER BY id DESC LIMIT 1;

result := db.First(&user)
result.RowsAffected // returns count of records found
result.Error        // returns error or nil

// check error ErrRecordNotFound
errors.Is(result.Error, gorm.ErrRecordNotFound)
```

!!! warning ErrRecordNotFound エラーを避けたい場合は、db.Limit(1).Find(&user)のように、Find を 使用することができます。
    Find メソッドは struct と slice のどちらも受け取ることができます
!!! warning 単一のオブジェクトに対して制限なしでFindを使用する db.Find(&user) は、テーブル全体をクエリし、最初のオブジェクトのみを返します。

First メソッドと Last メソッドは、プライマリ・キーで並べ替えられた最初と最後のレコードをそれぞれ検索します。
- これらのメソッドは、出力先構造体へのポインタが引数としてメソッドに渡されるか、db.Model()を使用してモデルが指定された場合にのみ動作します。
  - さらに、関連するモデルに主キーが定義されていない場合、モデルは最初のフィールドで並べ替えられます。
```go
var user User
var users []User

// 宛先構造体が SELECT * FROM `users` ORDER BY `users`.`id` LIMIT 1 で渡されるので、動作する。
db.First(&user)

// モデルが`db.Model()`で指定されているので動作する。
result := map[string]interface{}{}
db.Model(&User{}).First(&result)
// SELECT * FROM `users` ORDER BY `users`.`id` LIMIT 1

// これは動かない
result := map[string]interface{}{}
db.Table("users").First(&result)

// works with Take
result := map[string]interface{}{}
db.Table("users").Take(&result)

// 主キーが定義されていない場合、結果は最初のフィールド（つまり`コード`）の順に並べられる。
type Language struct {
  Code string
  Name string
}
db.First(&Language{})
// SELECT * FROM `languages` ORDER BY `languages`.`code` LIMIT 1
```
# プライマリキーでオブジェクトを取得する
主キーが数値の場合は、インライン条件を使って主キーを使ってオブジェクトを取り出すことができます。文字列を扱う場合は、SQLインジェクションを避けるために細心の注意が必要です。
```go
db.First(&user, 10)
// SELECT * FROM users WHERE id = 10;

db.First(&user, "10")
// SELECT * FROM users WHERE id = 10;

db.Find(&users, []int{1,2,3})
// SELECT * FROM users WHERE id IN (1,2,3);
If the primary key is a string (for example, like a uuid), the query will be written as follows:

db.First(&user, "id = ?", "1b74413f-f3b8-409f-ac47-e8c062e3472a")
// SELECT * FROM users WHERE id = "1b74413f-f3b8-409f-ac47-e8c062e3472a";
```
デスティネーション・オブジェクトがプライマリ値を持つ場合、プライマリ・キーは、例えば、コンディションを構築するために使用される：
```go
var user = User{ID: 10}
db.First(&user)
// SELECT * FROM users WHERE id = 10;

var result User
db.Model(User{ID: 10}).First(&result)
// SELECT * FROM users WHERE id = 10;
```
!!! warning NOTE: gorm.DeletedAt のような gorm 固有のフィールドタイプを使用する場合は、オブジェクトを取得するために別のクエリが実行されます。
```go
type User struct {
  ID           string `gorm:"primarykey;size:16"`
  Name         string `gorm:"size:24"`
  DeletedAt    gorm.DeletedAt `gorm:"index"`
}

var user = User{ID: 15}
db.First(&user)
//  SELECT * FROM `users` WHERE `users`.`id` = '15' AND `users`.`deleted_at` IS NULL ORDER BY `users`.`id` LIMIT 1
```
# すべてのオブジェクトを取得する(全件取得)
```go
// Get all records
result := db.Find(&users)
// SELECT * FROM users;

result.RowsAffected // returns found records count, equals `len(users)`
result.Error        // returns error
```
# 条件検索
## 文字列条件
```go
// 最初に一致したレコードを取得
db.Where("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE name = 'jinzhu' ORDER BY id LIMIT 1;

//一致するすべてのレコードを取得する
db.Where("name <> ?", "jinzhu").Find(&users)
// SELECT * FROM users WHERE name <> 'jinzhu';

// IN
db.Where("name IN ?", []string{"jinzhu", "jinzhu 2"}).Find(&users)
// SELECT * FROM users WHERE name IN ('jinzhu','jinzhu 2');

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
// SELECT * FROM users WHERE name LIKE '%jin%';

// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;

// Time
db.Where("updated_at > ?", lastWeek).Find(&users)
// SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';

// BETWEEN
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
// SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';
```
!!! warning オブジェクトの主キーが設定されている場合、条件クエリは主キーの値を対象とせず、'and'条件として使用する。
    ```go
    var user = User{ID: 10}
    db.Where("id = ?", 20).First(&user)
    // SELECT * FROM users WHERE id = 10 and id = 20 ORDER BY id ASC LIMIT 1
    ```
    このクエリではレコードが見つからないというエラーが発生します。
    - したがって、user のような変数を使用してデータベースから新しい値を取得する前に、id のような主キー属性を nil に設定してください。

## 構造体 & マップでの条件指定
```go
// Struct
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 ORDER BY id LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;

// Slice of primary keys
db.Where([]int64{20, 21, 22}).Find(&users)
// SELECT * FROM users WHERE id IN (20, 21, 22);
```
!!! warning NOTE: フィールドの値が0や''、falseなどのゼロの場合は、クエリ条件には使用されません
    ```go
    db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
    // SELECT * FROM users WHERE name = "jinzhu";
    ```

!!! info クエリー条件にゼロの値を含めるには、マップを使用すること
    ```go
    db.Where(map[string]interface{}{"Name": "jinzhu", "Age": 0}).Find(&users)
    // SELECT * FROM users WHERE name = "jinzhu" AND age = 0;
    ```
## 構造体の検索フィールドを指定する
構造体を使用して検索する場合は、`Where()` に関連するフィールド名または dbname を渡すことで、クエリ条件に使用する構造体の特定の値を指定できます、
```go
db.Where(&User{Name: "jinzhu"}, "name", "Age").Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;

db.Where(&User{Name: "jinzhu"}, "Age").Find(&users)
// SELECT * FROM users WHERE age = 0;
```
## インライン条件
クエリー条件は、Whereと同様にFirstやFindのようなメソッドにインライン化できる。
```go
// 非整数型の主キーで取得
db.First(&user, "id = ?", "string_primary_key")
// SELECT * FROM users WHERE id = 'string_primary_key';

// Plain SQL
db.Find(&user, "name = ?", "jinzhu")
// SELECT * FROM users WHERE name = "jinzhu";

db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

// Struct
db.Find(&users, User{Age: 20})
// SELECT * FROM users WHERE age = 20;

// Map
db.Find(&users, map[string]interface{}{"age": 20})
// SELECT * FROM users WHERE age = 20;
```
## Not条件
ビルドNOT条件、Whereと似た働きをする
```go
db.Not("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE NOT name = "jinzhu" ORDER BY id LIMIT 1;

// Not In
db.Not(map[string]interface{}{"name": []string{"jinzhu", "jinzhu 2"}}).Find(&users)
// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");

// Struct
db.Not(User{Name: "jinzhu", Age: 18}).First(&user)
// SELECT * FROM users WHERE name <> "jinzhu" AND age <> 18 ORDER BY id LIMIT 1;

// 主キーのスライス内にない
db.Not([]int64{1,2,3}).First(&user)
// SELECT * FROM users WHERE id NOT IN (1,2,3) ORDER BY id LIMIT 1;
OR条件
db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
// SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';

// Struct
db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2", Age: 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);

// Map
db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2", "age": 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);
```
## 特定のフィールドのみ選択
Selectではデータベースから取得したいフィールドを指定することができます。
- 指定しない場合、GORMはデフォルトで全てのフィールドを選択します。
```go
db.Select("name", "age").Find(&users)
// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
// SELECT COALESCE(age,'42') FROM users;
```
!!! info 詳細は[高度なクエリ](https://gorm.io/ja_JP/docs/advanced_query.html#smart_select)を参照
## フィールドの選択
>[高度なクエリ](https://gorm.io/ja_JP/docs/advanced_query.html#smart_select)

GORMでは Select で選択するフィールド指定することができます。
- アプリケーションでこれを頻繁に使用する場合は、特定のフィールドを自動的に選択できる、用途に適した構造体を定義するとよい
```go
type User struct {
  ID     uint
  Name   string
  Age    int
  Gender string
  // hundreds of fields
}

type APIUser struct {
  ID   uint
  Name string
}

// クエリー時に `id`, `name` を自動的に選択する。
db.Model(&User{}).Limit(10).Find(&APIUser{})
// SELECT `id`, `name` FROM `users` LIMIT 10
```
!!! warning 注意 QueryFields モードを有効にすると、モデルのすべてのフィールド名を選択するようになる
    ```go
    db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
    QueryFields: true,
    })

    db.Find(&user)
    // SELECT `users`.`name`, `users`.`age`, ... FROM `users` // with this option

    // Session Mode
    db.Session(&gorm.Session{QueryFields: true}).Find(&user)
    // SELECT `users`.`name`, `users`.`age`, ... FROM `users`   
    ```
# 順序
データベースからレコードを取得する際の順序を指定する。
```go
db.Order("age desc, name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

// Multiple orders
db.Order("age desc").Order("name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

db.Clauses(clause.OrderBy{
  Expression: clause.Expr{SQL: "FIELD(id,?)", Vars: []interface{}{[]int{1, 2, 3}}, WithoutParentheses: true},
}).Find(&User{})
// SELECT * FROM users ORDER BY FIELD(id,1,2,3)
```
# リミット＆オフセット
Limit 最大取得レコード数 Offset レコードを返し始める前にスキップするレコード数を指定する。
```go
db.Limit(3).Find(&users)
// SELECT * FROM users LIMIT 3;

// Cancel limit condition with -1
db.Limit(10).Find(&users1).Limit(-1).Find(&users2)
// SELECT * FROM users LIMIT 10; (users1)
// SELECT * FROM users; (users2)

db.Offset(3).Find(&users)
// SELECT * FROM users OFFSET 3;

db.Limit(10).Offset(5).Find(&users)
// SELECT * FROM users OFFSET 5 LIMIT 10;

// Cancel offset condition with -1
db.Offset(10).Find(&users1).Offset(-1).Find(&users2)
// SELECT * FROM users OFFSET 10; (users1)
// SELECT * FROM users; (users2)
```
ページネーターの作り方については、ページネーションを参照のこと。

# Group By & Having
```go
type result struct {
  Date  time.Time
  Total int
}

db.Model(&User{}).Select("name, sum(age) as total").Where("name LIKE ?", "group%").Group("name").First(&result)
// SELECT name, sum(age) as total FROM `users` WHERE name LIKE "group%" GROUP BY `name` LIMIT 1

db.Model(&User{}).Select("name, sum(age) as total").Group("name").Having("name = ?", "group").Find(&result)
// SELECT name, sum(age) as total FROM `users` GROUP BY `name` HAVING name = "group"

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Rows()
defer rows.Close()
for rows.Next() {
  ...
}

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Rows()
defer rows.Close()
for rows.Next() {
  ...
}

type Result struct {
  Date  time.Time
  Total int64
}
db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Scan(&results)
```
# 特徴的
モデルから明確な値を選択する
```go
db.Distinct("name", "age").Order("name, age desc").Find(&results)
```
ディスティンクトはプラックとカウントでも機能する
# Joins（結合）
結合条件の指定
```go
type result struct {
  Name  string
  Email string
}

db.Model(&User{}).Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&result{})
// SELECT users.name, emails.email FROM `users` left join emails on emails.user_id = users.id

rows, err := db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Rows()
for rows.Next() {
  ...
}

db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&results)

//パラメータによる多重結合
db.Joins("JOIN emails ON emails.user_id = users.id AND emails.email = ?", "jinzhu@example.org").Joins("JOIN credit_cards ON credit_cards.user_id = users.id").Where("credit_cards.number = ?", "411111111111").Find(&user)
```
# プリロードに結合
ジョインは、単一のSQLで熱心な関連付けをロードするために使用することができます、
```go
db.Joins("Company").Find(&users)
// SELECT `users`.`id`,`users`.`name`,`users`.`age`,`Company`.`id` AS `Company__id`,`Company`.`name` AS `Company__name` FROM `users` LEFT JOIN `companies` AS `Company` ON `users`.`company_id` = `Company`.`id`;

// inner join
db.InnerJoins("Company").Find(&users)
// SELECT `users`.`id`,`users`.`name`,`users`.`age`,`Company`.`id` AS `Company__id`,`Company`.`name` AS `Company__name` FROM `users` INNER JOIN `companies` AS `Company` ON `users`.`company_id` = `Company`.`id`;
```
## Join with conditions
```go
db.Joins("Company", db.Where(&Company{Alive: true})).Find(&users)
// SELECT `users`.`id`,`users`.`name`,`users`.`age`,`Company`.`id` AS `Company__id`,`Company`.`name` AS `Company__n
```
## 導出表の結合
Joinsを使用して派生テーブルを結合することもできます。
```go
type User struct {
    Id  int
    Age int
}

type Order struct {
    UserId     int
    FinishedAt *time.Time
}

query := db.Table("order").Select("MAX(order.finished_at) as latest").Joins("left join user user on order.user_id = user.id").Where("user.age > ?", 18).Group("order.user_id")
db.Model(&Order{}).Joins("join (?) q on order.finished_at = q.latest", query).Scan(&results)
// SELECT `order`.`user_id`,`order`.`finished_at` FROM `order` join (SELECT MAX(order.finished_at) as latest FROM `order` left join user user on order.user_id = user.id WHERE user.age > 18 GROUP BY `order`.`user_id`) q on order.finished_at = q.latest
```
# Scan
構造体への結果のスキャンは、Findの使い方と似ています。
```go
type Result struct {
  Name string
  Age  int
}

var result Result
db.Table("users").Select("name", "age").Where("name = ?", "Antonio").Scan(&result)

// Raw SQL
db.Raw("SELECT name, age FROM users WHERE name = ?", "Antonio").Scan(&result)
```













