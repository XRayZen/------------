# timeパッケージで日時操作(比較, 差分, フォーマット)
>[参考](https://www.wakuwakubank.com/posts/781-go-time/)

Goの標準パッケージであるtimeパッケージについて動作確認します。「日時の加算減算」「月初、月末の算出」「他の日時との比較」「スリープ処理」など利用頻度の高い処理について、どのように実装するか確認していきます。

Table of Contents
=================
- [timeパッケージで日時操作(比較, 差分, フォーマット)](#timeパッケージで日時操作比較-差分-フォーマット)
- [Table of Contents](#table-of-contents)
- [time.Time型の変数を取得( Now Date Parse Unix )](#timetime型の変数を取得-now-date-parse-unix-)
- [日時の加算減算( Add AddDate )](#日時の加算減算-add-adddate-)
- [月初、月末](#月初月末)
- [他の日時との差分( Sub )](#他の日時との差分-sub-)
- [他の日時との比較( Equal After Before )](#他の日時との比較-equal-after-before-)
- [Unixタイムスタンプ(Unix UnixNano)](#unixタイムスタンプunix-unixnano)
- [フォーマット( Format )](#フォーマット-format-)
- [Sleep](#sleep)

# time.Time型の変数を取得( Now Date Parse Unix )
timeパッケージの Now関数 Date関数 Parse関数 Unix関数を利用して、time.Time型のインスタンスを取得できます。
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t1 := time.Now()
	fmt.Println(t1)

	t2 := time.Date(2020, time.December, 10, 23, 1, 10, 0, time.UTC)
	fmt.Println(t2)

	t3 := time.Date(2020, time.December, 10, 23, 1, 10, 0, time.Local)
	fmt.Println(t3)

	t4, _ := time.Parse(time.RFC3339, "2020-12-02T15:04:05Z")
	fmt.Println(t4)

	t5, _ := time.Parse(time.RFC3339, "2020-12-02T15:04:05+09:00")
	fmt.Println(t5)

	t6 := time.Unix(1606889045, 0)
	fmt.Println(t6)
}
```
>出力
```
2020-12-20 20:15:51.383788 +0900 JST m=+0.000125260
2020-12-10 23:01:10 +0000 UTC
2020-12-10 23:01:10 +0900 JST
2020-12-02 15:04:05 +0000 UTC
2020-12-02 15:04:05 +0900 JST
2020-12-02 15:04:05 +0900 JST
```
# 日時の加算減算( Add AddDate )
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t1, _ := time.Parse(time.RFC3339, "2020-10-15T15:00:00+09:00")
	fmt.Println(t1)
	fmt.Println("-----------------------")
	fmt.Println(t1.Add(time.Second * 10))
	fmt.Println(t1.Add(time.Second * -10))
	fmt.Println("-----------------------")
	fmt.Println(t1.Add(time.Minute * 10))
	fmt.Println(t1.Add(time.Minute * -10))
	fmt.Println(t1.Add(time.Minute * time.Duration(10)))
	fmt.Println("-----------------------")
	fmt.Println(t1.Add(time.Hour * 10))
	fmt.Println(t1.Add(time.Hour * -10))
	fmt.Println("-----------------------")
	fmt.Println(t1.Add(time.Hour * 24 * 10))
	fmt.Println(t1.Add(time.Hour * 24 * -10))
}
```
>出力
```
2020-10-15 15:00:00 +0900 JST
-----------------------
2020-10-15 15:00:10 +0900 JST
2020-10-15 14:59:50 +0900 JST
-----------------------
2020-10-15 15:10:00 +0900 JST
2020-10-15 14:50:00 +0900 JST
-----------------------
2020-10-16 01:00:00 +0900 JST
2020-10-15 05:00:00 +0900 JST
-----------------------
2020-10-25 15:00:00 +0900 JST
2020-10-05 15:00:00 +0900 JST
```
日付以上の単位で加算減算したい場合、AddDateメソッド を活用できます。
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t1, _ := time.Parse(time.RFC3339, "2020-10-15T15:00:00+09:00")
	fmt.Println(t1)
	fmt.Println("-----------------------")
	fmt.Println(t1.AddDate(0, 0, 1))
	fmt.Println(t1.AddDate(0, 0, -1))
	fmt.Println("-----------------------")
	fmt.Println(t1.AddDate(0, 1, 0))
	fmt.Println(t1.AddDate(0, -1, 0))
	fmt.Println("-----------------------")
	fmt.Println(t1.AddDate(1, 0, 0))
	fmt.Println(t1.AddDate(-1, 0, 0))
}
```
>出力
```
2020-10-15 15:00:00 +0900 JST
-----------------------
2020-10-16 15:00:00 +0900 JST
2020-10-14 15:00:00 +0900 JST
-----------------------
2020-11-15 15:00:00 +0900 JST
2020-09-15 15:00:00 +0900 JST
-----------------------
2021-10-15 15:00:00 +0900 JST
2019-10-15 15:00:00 +0900 JST
```
# 月初、月末
月初は、Date関数で求めることができます。
月末は、月初の変数をもとに AddDateメソッド で求めることができます。
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t1, _ := time.Parse(time.RFC3339, "2020-12-02T15:04:05+09:00")
	fmt.Println(t1)

	// 月初
	t2 := time.Date(t1.Year(), t1.Month(), 1, 0, 0, 0, 0, time.Local)
	fmt.Println(t2)

	// 月末
	t3 := t2.AddDate(0, 1, -1)
	fmt.Println(t3)

	t4 := time.Date(2021, 1, 1, 0, 0, 0, 0, time.Local).AddDate(0,0,-1)
	fmt.Println(t4)
}
```
>出力
```
2020-12-02 15:04:05 +0900 JST
2020-12-01 00:00:00 +0900 JST
2020-12-31 00:00:00 +0900 JST
2020-12-31 00:00:00 +0900 JST
```
# 他の日時との差分( Sub )
Subメソッドを利用すると他のtime.Time型の変数との差分を求めることができます。
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t1, _ := time.Parse(time.RFC3339, "2020-10-01T15:00:00+09:00")
	t2, _ := time.Parse(time.RFC3339, "2020-12-01T15:00:00+09:00")
	t3, _ := time.Parse(time.RFC3339, "2020-12-03T15:00:00+09:00")
	t4, _ := time.Parse(time.RFC3339, "2020-12-03T20:00:00+09:00")

	diff1 := t2.Sub(t1)
	fmt.Println(diff1)
	fmt.Println(diff1.Hours())

	fmt.Println(t3.Sub(t2))
	fmt.Println(t4.Sub(t3))
}
```
>出力
```
1464h0m0s
1464
48h0m0s
5h0m0s
```
# 他の日時との比較( Equal After Before )
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t1, _ := time.Parse(time.RFC3339, "2020-11-01T15:00:00+09:00")
	t2, _ := time.Parse(time.RFC3339, "2020-11-01T15:00:00+09:00")
	t3, _ := time.Parse(time.RFC3339, "2020-11-01T14:00:00+09:00")
	t4, _ := time.Parse(time.RFC3339, "2020-11-01T16:00:00+09:00")

	fmt.Println(t1.Equal(t2))
	fmt.Println(t1.After(t2))
	fmt.Println(t1.Before(t2))
	fmt.Println("-----------------------")
	fmt.Println(t1.Equal(t3))
	fmt.Println(t1.After(t3))
	fmt.Println(t1.Before(t3))
	fmt.Println("-----------------------")
	fmt.Println(t1.Equal(t4))
	fmt.Println(t1.After(t4))
	fmt.Println(t1.Before(t4))
}
```
>出力
```
true
false
false
-----------------------
false
true
false
-----------------------
false
false
true
```
# Unixタイムスタンプ(Unix UnixNano)
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t1, _ := time.Parse(time.RFC3339, "2020-12-02T15:04:05+09:00")

	fmt.Println(t1.Unix())
	fmt.Println(t1.UnixNano())
}
```
>出力
```
1606889045
1606889045000000000
```
# フォーマット( Format )
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t1, _ := time.Parse(time.RFC3339, "2020-12-02T20:04:05+09:00")
	fmt.Println(t1.Format(time.RFC3339))
	fmt.Println(t1.Format("2006-01-02"))
	fmt.Println(t1.Format("2006-01-02 03:04:05"))
	fmt.Println(t1.Format("3:4:5"))
	fmt.Println(t1.Format("03:04:05"))
	fmt.Println(t1.Format("15:04:05"))
}
```
>出力
```
2020-12-02T20:04:05+09:00
2020-12-02
2020-12-02 08:04:05
8:4:5
08:04:05
20:04:05
```
フォーマットの指定方法はわかりずらくなっています。
このようなフォーマット指定になった経緯は下記ページで説明されていました。
- [Goのtimeパッケージのリファレンスタイム（2006年1月2日）は何の日？](https://qiita.com/ruiu/items/5936b4c3bd6eb487c182)
# Sleep
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println(time.Now())
	time.Sleep(10 * time.Second)
	fmt.Println(time.Now())
}
```
>出力
```
2020-12-20 22:04:53.745756 +0900 JST m=+0.000088885
2020-12-20 22:05:03.746833 +0900 JST m=+10.000865869
```

































