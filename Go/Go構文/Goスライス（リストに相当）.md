# スライス操作(要素の追加・削除, ソート, 他のスライスと結合)
>[参考記事](https://www.wakuwakubank.com/posts/782-go-slice/)

Goでスライス操作に必要な知識をまとめました。
- 「要素の追加・削除」「並び替え」「他のスライスと結合」「長さ(len), 容量(cap)の取得」といった処理を確認します。

Table of Contents
=================
- [スライス操作(要素の追加・削除, ソート, 他のスライスと結合)](#スライス操作要素の追加削除-ソート-他のスライスと結合)
- [Table of Contents](#table-of-contents)
- [要素値取得](#要素値取得)
- [rangeで1要素ずつ処理](#rangeで1要素ずつ処理)
- [長さと容量｜len, cap](#長さと容量len-cap)
- [makeでスライスを生成](#makeでスライスを生成)
- [strings.Joinで結合](#stringsjoinで結合)
- [要素変更](#要素変更)
  - [末尾に追加](#末尾に追加)
  - [先頭に追加](#先頭に追加)
  - [末尾を削除](#末尾を削除)
  - [先頭を削除](#先頭を削除)
  - [他のスライスと結合](#他のスライスと結合)
- [copy](#copy)
- [並び替え(ソート)](#並び替えソート)
  - [Int](#int)
  - [Struct](#struct)

# 要素値取得
スライスの要素値を取得するには、インデックスを指定します。
```go
package main

import "fmt"

func main() {
	s1 := []int{1, 2, 3, 4, 5, 6}
	fmt.Println(s1)
	fmt.Println(s1[2])
	fmt.Println(s1[3:])
	fmt.Println(s1[:3])
	fmt.Println(s1[2:4])
	fmt.Println(s1[:])
}
```
>出力
```
[1 2 3 4 5 6]
3
[4 5 6]
[1 2 3]
[3 4]
[1 2 3 4 5 6]
```
# rangeで1要素ずつ処理
```go
package main

import "fmt"

func main() {
	s1 := []int{1, 2, 3}
	for i, v := range s1 {
		fmt.Printf("index: %d value: %d type: %T\n", i, v, v)
	}

	s2 := []interface{}{"abc", 111, true}
	for i, v := range s2 {
		fmt.Printf("index: %d value: %v type: %T\n", i, v, v)
	}

	s3 := [][]string{
		{"a", "b", "c"},
		{"d", "e", "f"},
		{"g", "h", "i"},
	}
	for i, v := range s3 {
		fmt.Printf("index: %d value: %v type: %T\n", i, v, v)
	}
}
```
>出力
```
index: 0 value: 1 type: int
index: 1 value: 2 type: int
index: 2 value: 3 type: int
index: 0 value: abc type: string
index: 1 value: 111 type: int
index: 2 value: true type: bool
index: 0 value: [a b c] type: []string
index: 1 value: [d e f] type: []string
index: 2 value: [g h i] type: []string
```
# 長さと容量｜len, cap
```go
package main

import "fmt"

func main() {
	s1 := []int{1, 2, 3, 4, 5}
	fmt.Printf("len:%d cap:%d\n", len(s1), cap(s1))

	s2 := []interface{}{"abc", 111, true}
	fmt.Printf("len:%d cap:%d\n", len(s2), cap(s2))

	s3 := [][]string{
		{"a", "b", "c"},
		{"d", "e", "f"},
		{"g", "h", "i"},
	}
	fmt.Printf("len:%d cap:%d\n", len(s3), cap(s3))
}
```
>出力
```
len:5 cap:5
len:3 cap:3
len:3 cap:3
```
# makeでスライスを生成
```go
package main

import "fmt"

type SampleUser struct {
	i int
	s string
	b bool
}

func main() {
	// 要素数に5を指定、容量は未指定
	i1 := make([]int, 5)
	fmt.Printf("len:%d cap:%d value:%v\n", len(i1), cap(i1), i1)

	// 要素数に0を指定、容量に5を指定
	i2 := make([]int, 0, 5)
	fmt.Printf("len:%d cap:%d value:%v\n", len(i2), cap(i2), i2)

	// 要素数に2を指定、容量に5を指定
	i3 := make([]int, 2, 5)
	fmt.Printf("len:%d cap:%d value:%v\n", len(i3), cap(i3), i3)

	// 要素数に5を指定、容量は未指定
	i4 := make([]SampleUser, 5)
	fmt.Printf("len:%d cap:%d value:%v\n", len(i4), cap(i4), i4)

	// 要素数に0を指定、容量に5を指定
	i5 := make([]SampleUser, 0, 5)
	fmt.Printf("len:%d cap:%d value:%v\n", len(i5), cap(i5), i5)
}
```
容量が未指定のとき、要素数が容量になります。要素数が0以外のとき、初期化された値が格納されています。
>出力
```
len:5 cap:5 value:[0 0 0 0 0]
len:0 cap:5 value:[]
len:2 cap:5 value:[0 0]
len:5 cap:5 value:[{0  false} {0  false} {0  false} {0  false} {0  false}]
len:0 cap:5 value:[]
```
# strings.Joinで結合
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	s1 := []string{"a", "b", "c"}
	fmt.Println(strings.Join(s1[:], ","))
}
```
>出力
```
a,b,c
```
# 要素変更
## 末尾に追加
```go
package main

import (
	"fmt"
)

func main() {
	s1 := []string{"a", "b", "c"}
	s2 := append(s1, "d")

	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s2), cap(s2), s2)
}
```
>出力
```
len:3 cap:3 v:[a b c]
len:4 cap:6 v:[a b c d]
```
## 先頭に追加
```go
package main

import (
	"fmt"
)

func main() {
	s1 := []string{"a", "b", "c"}
	s2 := append([]string{"d"}, s1[0:]...)

	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s2), cap(s2), s2)
}
```
>出力
```
len:3 cap:3 v:[a b c]
len:4 cap:4 v:[d a b c]
```
## 末尾を削除
```go
package main

import (
	"fmt"
)

func main() {
	s1 := []string{"a", "b", "c"}
	s2 := s1[:len(s1)-1]

	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s2), cap(s2), s2)
}
```
>出力
```
len:3 cap:3 v:[a b c]
len:2 cap:3 v:[a b]
```
## 先頭を削除
```go
package main

import (
	"fmt"
)

func main() {
	s1 := []string{"a", "b", "c"}
	s2 := s1[1:]

	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s2), cap(s2), s2)
}
```
>出力
```
len:3 cap:3 v:[a b c]
len:2 cap:2 v:[b c]
```
## 他のスライスと結合
```go
package main

import (
	"fmt"
)

func main() {
	s1 := []string{"a", "b", "c"}
	s2 := []string{"d", "e", "f"}
	s3 := append(s1, s2...)

	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s2), cap(s2), s2)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s3), cap(s3), s3)
}
```
>出力
```
len:3 cap:3 v:[a b c]
len:3 cap:3 v:[d e f]
len:6 cap:6 v:[a b c d e f]
```
# copy
```go
package main

import (
	"fmt"
)

func main() {
	s1 := []int{1, 2, 3, 4, 5, 6}
	s2 := make([]int, 3)
	s3 := make([]int, 8)
	s4 := make([]int, len(s1))
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s2), cap(s2), s2)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s3), cap(s3), s3)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s4), cap(s4), s4)
	fmt.Println()

	fmt.Println("-- COPY1 --")
	n1 := copy(s2, s1)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s2), cap(s2), s2)
	fmt.Println(n1)

	fmt.Println("-- COPY2 --")
	n2 := copy(s3, s1)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s3), cap(s3), s3)
	fmt.Println(n2)

	fmt.Println("-- COPY3 --")
	n3 := copy(s4, s1)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s4), cap(s4), s4)
	fmt.Println(n3)
}
```
>出力
```
len:6 cap:6 v:[1 2 3 4 5 6]
len:3 cap:3 v:[0 0 0]
len:8 cap:8 v:[0 0 0 0 0 0 0 0]
len:6 cap:6 v:[0 0 0 0 0 0]

-- COPY1 --
len:3 cap:3 v:[1 2 3]
3
-- COPY2 --
len:8 cap:8 v:[1 2 3 4 5 6 0 0]
6
-- COPY3 --
len:6 cap:6 v:[1 2 3 4 5 6]
6
```
# 並び替え(ソート)
## Int
```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	s1 := []int{41, 23, 17, 39, 53, 12, 83, 9}
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)

	// 昇順
	sort.Sort(sort.IntSlice(s1))
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)

	// 降順
	sort.Sort(sort.Reverse(sort.IntSlice(s1)))
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
}
```
>出力
```
len:8 cap:8 v:[41 23 17 39 53 12 83 9]
len:8 cap:8 v:[9 12 17 23 39 41 53 83]
len:8 cap:8 v:[83 53 41 39 23 17 12 9]
```
## Struct
```go
package main

import (
	"fmt"
	"sort"
)

type User struct {
	Name  string
	Score int
}

func main() {
	u := []User{
		{Name: "aaa", Score: 41},
		{Name: "bbb", Score: 23},
		{Name: "ccc", Score: 17},
		{Name: "ddd", Score: 39},
		{Name: "eee", Score: 53},
		{Name: "fff", Score: 12},
		{Name: "ggg", Score: 83},
		{Name: "hhh", Score: 9},
	}

	fmt.Printf("len:%d cap:%d v:%v\n", len(u), cap(u), u)

	// Score 昇順
	sort.Slice(u, func(i, j int) bool { return u[i].Score < u[j].Score })
	fmt.Printf("len:%d cap:%d v:%v\n", len(u), cap(u), u)

	// Score 降順
	sort.Slice(u, func(i, j int) bool { return u[i].Score > u[j].Score })
	fmt.Printf("len:%d cap:%d v:%v\n", len(u), cap(u), u)
}
```
>出力
```
len:8 cap:8 v:[{aaa 41} {bbb 23} {ccc 17} {ddd 39} {eee 53} {fff 12} {ggg 83} {hhh 9}]
len:8 cap:8 v:[{hhh 9} {fff 12} {ccc 17} {bbb 23} {ddd 39} {aaa 41} {eee 53} {ggg 83}]
len:8 cap:8 v:[{ggg 83} {eee 53} {aaa 41} {ddd 39} {bbb 23} {ccc 17} {fff 12} {hhh 9}]
```



































