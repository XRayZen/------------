# Update
>[GORM](https://gorm.io/ja_JP/docs/update.html)
>[Zenn](https://zenn.dev/a_ichi1/articles/4b113d4c46857a#crud)

updateは主に3つのメソッドがあります。

# save
upsertになる。
既存のものがあれば上書き、なければ追加となります。
## 使い方
先に単体取得をしておいて、構造体とマッピングしておく。
その構造体の中身を更新したい値に変更する。
更新した構造体をsaveに渡すと、更新される。
その構造体の中にidやプライマリーキーが入っていないと、新しいレコードが作成される。
## update
1つのカラムだけを更新する。
### updates
複数のカラムを更新する。
```go
// 更新(upsert)
func save(db *gorm.DB) {
	// 構造体にidが無い場合はinsertされる
	user1 := User{}
	user1.Name = "花子"
	result1 := db.Save(&user1)
	if result1.Error != nil {
		log.Fatal(result1.Error)
	}
	fmt.Println("count:", result1.RowsAffected)
	fmt.Println("user1:", user1)

	// 先にユーザーを取得する
	user2 := User{}
	db.First(&user2)

	// 構造体にidがある場合はupdateされる
	user2.Name = "たけし"
	result2 := db.Save(&user2)
	if result2.Error != nil {
		log.Fatal(result2.Error)
	}
	fmt.Println("count:", result2.RowsAffected)
	fmt.Println("user2:", user2)
}
```
### 出力結果
```go
count: 1
user1: {{5 2022-05-24 13:34:18.663 +0900 JST 2022-05-24 13:34:18.663 +0900 JST {0001-01-01 00:00:00 +0000 UTC false}} 花子 0 false}
count: 1
user2: {{1 2022-05-23 04:33:26.367 +0000 UTC 2022-05-24 13:34:18.688 +0900 JST {00
```






