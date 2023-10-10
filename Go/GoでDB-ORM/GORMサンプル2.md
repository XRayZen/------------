# GORMというORMが超絶便利だった件
>[参考元](https://medium.com/@taka.abc.hiko/gorm%E3%81%A8%E3%81%84%E3%81%86orm%E3%81%8C%E8%B6%85%E7%B5%B6%E4%BE%BF%E5%88%A9%E3%81%A0%E3%81%A3%E3%81%9F%E4%BB%B6-8d279489c38f)

>追加参考
>http://gorm.io/docs/
>http://doc.gorm.io/
>https://godoc.org/github.com/jinzhu/gorm


# マイグレーション
マイグレーションは、定義したstructをAutoMigrateの引数に渡すことで、それに対応するテーブルの作成を行う。
```go
// Model 
type Model struct {
   ID        uint `gorm:"primary_key" json:"id"`
   CreatedAt time.Time
   UpdatedAt time.Time
   DeletedAt *time.Time `sql:"index" json:"-"`
}

// 人物を表す構造体
type Person struct {
   Model
   Name string
   Age  int
}

func main() {
   
   db := GetDBConn()

   // Personテーブルの作成
   db.AutoMigrate(&Person{})

}
```

# SELECT
```go
// Model 
type Model struct {
   ID        uint `gorm:"primary_key" json:"id"`
   CreatedAt time.Time
   UpdatedAt time.Time
   DeletedAt *time.Time `sql:"index" json:"-"`
}

// 人物を表す構造体
type Person struct {
   Model
   Name string
   Age  int
}
var person Person
var persons []Person // 複数件取得する場合、構造体を配列にする

func main() {

   db := GetDBConn()

   // SELECT * FROM persons;
   db.Find(&persons)

   // SELECT * FROM persons ORDER BY id LIMIT 1;
   db.First(&person)

   // SELECT * FROM persons ORDER BY id DESC LIMIT 1;
   db.Last(&person)

   // SELECT * FROM persons WHERE id = 1;
   db.First(&person, 1)

   // SELECT * FROM persons WHERE name = 'Gopher';
   db.Where("name = ?", "Gopher").Find(&person)

   // SELECT * FROM persons WHERE name = 'Gopher' limit 1;
   db.Where("name = ?", "Gopher").First(&person)

   // SELECT * FROM persons WHERE name = 'Gopher' AND id >= 1;
   db.Where("name = ? AND age >= ?", "Gopher", 1).Find(&persons)

   // SELECT * FROM persons WHERE name = 'Gopher' OR age = 1;
   db.Where("name = ?", "Gopher").Or("age = ?", 1).Find(&persons)

   // SELECT * FROM persons WHERE name LIKE '%Go%';
   db.Where("name like ?", "%Go%").Find(&persons)

   // SELECT * FROM persons WHERE NOT(name = 'Gopher');
   db.Not("name = ?", "Gopher").First(&person)

}
```
# INSERT
```go
// Model 
type Model struct {
   ID        uint `gorm:"primary_key" json:"id"`
   CreatedAt time.Time
   UpdatedAt time.Time
   DeletedAt *time.Time `sql:"index" json:"-"`
}

// 人物を表す構造体
type Person struct {
   Model
   Name string
   Age  int
}

func main() {

   db := GetDBConn()

   var person = Person{Name: "Gopher", Age: 10}

   // INSERT INTO persons("name","age") values('Gopher',10);
   db.Create(&person)

}
```
# UPDATE
```go
// Model 
type Model struct {
   ID        uint `gorm:"primary_key" json:"id"`
   CreatedAt time.Time
   UpdatedAt time.Time
   DeletedAt *time.Time `sql:"index" json:"-"`
}

// 人物を表す構造体
type Person struct {
   Model
   Name string
   Age  int
}

func main() {

   db := GetDBConn()

   db.First(&person)

   person.Name = "ゴーファー"
   person.ID = 99

   // UPDATE persons SET name = 'ゴーファー', age = 99 updated_at = '2018-08-27 21:34:10' WHERE id=1;
   db.Save(&person)
}
```
# DELETE
```go
func main() {

   db := GetDBConn()

   // DELETE FROM persons WHERE id = 1;
   db.Delete(&person, 1)
   
}
```






























