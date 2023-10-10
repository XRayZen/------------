# データ型について動作確認(基本型・合成型・参照型・型変換-キャスト)
>[参考記事](https://www.wakuwakubank.com/posts/776-go-datatype/)

Goのデータ型には、基本型(数値型 / 浮動小数点型 / 論理値型 / 文字列型 / rune / byte)、合成型(配列 / 構造体)、参照型(Slice / Map / ポインタ / チャネル / 関数)といった型が存在します。
- ここでは、各データ型について簡単な処理を実行させて、動作確認をしていきます。

table of contents
=================
- [データ型について動作確認(基本型・合成型・参照型・型変換-キャスト)](#データ型について動作確認基本型合成型参照型型変換-キャスト)
- [table of contents](#table-of-contents)
- [基本型](#基本型)
  - [数値型 int](#数値型-int)
  - [浮動小数点 float](#浮動小数点-float)
  - [論理値型 bool](#論理値型-bool)
  - [文字列型 string](#文字列型-string)
  - [rune](#rune)
  - [byte](#byte)
- [合成型](#合成型)
  - [Array 配列](#array-配列)
  - [Struct(構造体)](#struct構造体)
- [参照型](#参照型)
  - [Slice](#slice)
  - [多次元Slice](#多次元slice)
  - [Map](#map)
  - [MapとStructの組み合わせ注意点( Cannot assign to xxx )](#mapとstructの組み合わせ注意点-cannot-assign-to-xxx-)
  - [多次元Map](#多次元map)
  - [ポインタ](#ポインタ)
  - [Channel チャネル](#channel-チャネル)
  - [関数](#関数)
- [interface{}](#interface)

# 基本型
## 数値型 int
```go
package main

import (
	"fmt"
)

func main() {
	var (
		i1 int    = 10  // 符号あり	実行アーキテクチャに依存(32bit or 64bit)
		i2 int    = 1e9 // 10の9乗(=1000000000)
		i3 int16  = 10  // 符号あり	最小:-32768                  最大:32767
		i4 int32  = 10  // 符号あり	最小:-2147483648             最大:2147483647 (約20億)
		i5 int64  = 10  // 符号あり	最小:-9223372036854775808    最大:9223372036854775807 (約900京)
		i6 uint   = 10  // 符号なし	実行アーキテクチャに依存(32bit or 64bit)
		i7 uint32 = 10  // 符号なし	最小:0                       最大:4294967295 (約40億)
	)
	fmt.Printf("%T %T %T %T %T %T %T\n", i1, i2, i3, i4, i5, i6, i7)
}
```
>出力
```
int int int16 int32 int64 uint uint32
```
## 浮動小数点 float
```go
package main

import (
	"fmt"
)

func main() {
	var (
		f32 float32 = 1.11111111111111111
		f64 float64 = 1.11111111111111111
	)
	fmt.Printf("%T\n", f32)
	fmt.Println(f32)
	fmt.Printf("%T\n", f64)
	fmt.Println(f64)
}
```
>出力
```
float32
1.1111112
float64
1.1111111111111112
```
## 論理値型 bool
```go
package main

import (
	"fmt"
)

func main() {
	var (
		t bool = true
		f bool = false
	)

	fmt.Printf("%T\n", t)
	fmt.Println(t)
	fmt.Printf("%T\n", f)
	fmt.Println(f)
}
```
>出力
```
bool
true
bool
false
```
## 文字列型 string
```go
package main

import (
	"fmt"
)

func main() {
	var (
		s1 string = "わくわくBank"   // ダブルクォーテーションで囲む

		// ヒアドキュメント(バッククォートで囲む)
		s2 string = `Hello
World`
	)

	fmt.Printf("%T\n", s1)
	fmt.Println(s1)
	fmt.Println(s2)
	fmt.Println(len(s1))
	for i, v := range s1 {
		// range構文を利用すると、rune単位で取得できる
		//   アルファベットは1文字1バイト
		//   日本語はほぼ1文字3バイト
		fmt.Printf("%T i:%v v:%v\n", v, i, v)
	}
	fmt.Printf("s[12]: %v\n", s1[12])
	fmt.Printf("[]rune(s): %v\n", []rune(s1))
	fmt.Printf("[]byte(s): %v\n", []byte(s1))
}
```
>出力
```
string
わくわくBank
Hello
World
16
int32 i:0 v:12431
int32 i:3 v:12367
int32 i:6 v:12431
int32 i:9 v:12367
int32 i:12 v:66
int32 i:13 v:97
int32 i:14 v:110
int32 i:15 v:107
s[12]: 66
[]rune(s): [12431 12367 12431 12367 66 97 110 107]
[]byte(s): [227 130 143 227 129 143 227 130 143 227 129 143 66 97 110 107]
```
## rune
```go
package main

import (
	"fmt"
)

func main() {
	var (
		r1 rune = 97
		r2 rune = 'a' // シングルクオートはRune型を扱う
	)
	fmt.Printf("%T\n", r1)
	fmt.Println(r1)
	fmt.Println(string(r1))

	fmt.Printf("%T\n", r2)
	fmt.Println(r2)
	fmt.Println(string(r2))
}
```
>出力
```
int32
97
a
int32
97
a
```
## byte
```go
package main

import (
	"fmt"
)

func main() {
	var (
		b byte = 97
	)
	fmt.Printf("%T\n", b)
	fmt.Println(b)
	fmt.Println(string(b))
}
```
>出力
```
uint8
97
a
```
# 合成型
## Array 配列
配列の要素数は固定です。要素の追加など行いたい場合 スライス を利用します。
```go
package main

import "fmt"

func main() {
	var array1 [2]string
	array1[0] = "aaa"
	array1[1] = "bbb"
	fmt.Printf("%T\n", array1)
	fmt.Println(array1[0], array1[1])
	fmt.Println(array1)
	fmt.Println(len(array1))
	fmt.Println("-----------------------")

	array2 := [4]int{1, 2, 3, 4}
	fmt.Printf("%T\n", array2)
	fmt.Println(array2)
	fmt.Println("-----------------------")

	array3 := [...]int{1, 2, 3, 4}
	fmt.Printf("%T\n", array3)
	fmt.Println(array3)
}
```
>出力
```
[2]string
aaa bbb
[aaa bbb]
2
-----------------------
[4]int
[1 2 3 4]
-----------------------
[4]int
[1 2 3 4]
```
## Struct(構造体)
```go
package main

import "fmt"

type User struct {
	ID    int
	Name  string
	age   int // 先頭小文字はprivate
	Score int
}

func main() {
	u1 := User{}
	fmt.Printf("%T\n", u1)
	fmt.Println(u1)

	u2 := User{1, "yamada", 30, 100}
	fmt.Printf("%T\n", u2)
	fmt.Println(u2)

	u2.Name = "suzuki"
	u2.age = 31 // 同一パッケージからは更新可能
	fmt.Println(u2)

	// 構造体ポインタを介して構造体内の値を更新
	p := &u2
	p.age = 32
	fmt.Printf("%T\n", p)
	fmt.Println(p)
	fmt.Println(u2)
}
```
>出力
```
main.User
{0  0 0}
main.User
{1 yamada 30 100}
{1 suzuki 31 100}
*main.User
&{1 suzuki 32 100}
{1 suzuki 32 100}
```
# 参照型
## Slice
cap(capacity=容量) の変更が発生するとき、メモリ領域の取り直しが行われるので処理コストが大きくなります。
```go
package main

import "fmt"

func main() {
	s1 := []int{1, 2, 3, 4, 5, 6, 7, 8}
	fmt.Printf("type: %T\n", s1)
    >> type: []int
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
    >> len:8 cap:8 v:[1 2 3 4 5 6 7 8]

	s1 = s1[:0]
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
    >> len:0 cap:8 v:[]

	s1 = s1[:2]
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
    >> len:2 cap:8 v:[1 2]

	s1 = s1[:4]
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
    >> len:4 cap:8 v:[1 2 3 4]

	s1 = s1[2:]
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
    >> len:2 cap:6 v:[3 4]

	s1 = s1[:4]
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
    >> len:4 cap:6 v:[3 4 5 6]
    

	s1 = append(s1, 1, 2)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
    >> len:6 cap:6 v:[3 4 5 6 1 2]

	s1 = append(s1, 3)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
    >> len:7 cap:12 v:[3 4 5 6 1 2 3]
	fmt.Println("----------------------------------------")

	// makeでsliceを生成
	s2 := make([]int, 5, 10)
	fmt.Printf("type: %T\n", s2)
    >> type: []int
	fmt.Printf("len:%d cap:%d v:%v\n", len(s2), cap(s2), s2)
    >> len:5 cap:10 v:[0 0 0 0 0]
	fmt.Println("----------------------------------------")

	s3 := make([]int, 5)
	fmt.Printf("type: %T\n", s3)
    >> type: []int
	fmt.Printf("len:%d cap:%d v:%v\n", len(s3), cap(s3), s3)
    >> len:5 cap:5 v:[0 0 0 0 0]
	fmt.Println("----------------------------------------")

	s4 := make([]int, 0, 10)
	fmt.Printf("type: %T\n", s4)
    >> type: []int
	fmt.Printf("len:%d cap:%d v:%v\n", len(s4), cap(s4), s4)
    >> len:0 cap:10 v:[]
}
```
## 多次元Slice
```go
package main

import "fmt"

func main() {
	s1 := [][]int{
		{1, 2, 3},
		{1, 2, 3, 4, 5, 6, 7, 8},
	}
	fmt.Printf("type: %T\n", s1)
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)

	s1 = append(s1, []int{1, 2})
	fmt.Printf("len:%d cap:%d v:%v\n", len(s1), cap(s1), s1)
}
```
>出力
```
type: [][]int
len:2 cap:2 v:[[1 2 3] [1 2 3 4 5 6 7 8]]
len:3 cap:4 v:[[1 2 3] [1 2 3 4 5 6 7 8] [1 2]]
```
## Map
```go
func main() {
	var m1 map[string]int
	fmt.Printf("%T\n", m1)

	fmt.Println("----------------------")
	fmt.Println(m1)
	// var宣言だけでは中身はnil
	fmt.Printf("m1 == nil: %v\n", m1 == nil)
	// 設定有無はlenで判定可能
	fmt.Printf("len(m1): %v\n", len(m1))

	fmt.Println("----------------------")
	// makeを使うことでメモリ領域がとられる
	m1 = make(map[string]int)
	fmt.Println(m1)
	fmt.Printf("m1 == nil: %v\n", m1 == nil)
	fmt.Printf("len(m1): %v\n", len(m1))
	fmt.Println("----------------------")

	m1["aaa"] = 1
	m1["bbb"] = 2
	m1["ccc"] = 3
	fmt.Println(m1)
	fmt.Printf("len(m1): %v\n", len(m1))
	fmt.Println("----------------------")

	delete(m1, "bbb")
	fmt.Println(m1)
	fmt.Println("----------------------")

	_, ok1 := m1["bbb"]
	fmt.Println(ok1)
	_, ok2 := m1["ccc"]
	fmt.Println(ok2)
}
```
>出力
```
map[string]int
----------------------
map[]
m1 == nil: true
len(m1): 0
----------------------
map[]
m1 == nil: false
len(m1): 0
----------------------
map[aaa:1 bbb:2 ccc:3]
len(m1): 3
----------------------
map[aaa:1 ccc:3]
----------------------
false
true
```
## MapとStructの組み合わせ注意点( Cannot assign to xxx )
MapにStructを格納した場合、Mapに格納したままStructの要素を更新することはできません。
```go
package main

import "fmt"

type sampleStruct struct {
	x int
}

func main() {
	sampleMap := make(map[int]sampleStruct, 1)
	sampleMap[0] = sampleStruct{x: 100}

	// sampleMap[0].x: 100
	fmt.Printf("sampleMap[0].x: %d\n", sampleMap[0].x)

	// 以下は「Cannot assign to sampleMap[0].x」になります。
	// sampleMap[0].x = 200

	// 以下のように一度取り出す必要があります。
	sm := sampleMap[0]
	sm.x = 200
	sampleMap[0] = sm
	
	// sampleMap[0].x: 200
	fmt.Printf("sampleMap[0].x: %d\n", sampleMap[0].x)
}
```
## 多次元Map
```go
package main

import "fmt"

func main() {
	var m2 map[string]map[string]int
	m2 = make(map[string]map[string]int)
	fmt.Printf("%T\n", m2)
	fmt.Println(m2)
	fmt.Println("----------------------")

	// 以下、エラーになります(panic: assignment to entry in nil map)。m2["aaa"]はnilのため。
	// m2["aaa"]["xxx"] = 1

	if _, ok := m2["aaa"]; !ok {
		m2["aaa"] = make(map[string]int)
		m2["aaa"]["xxx"] = 1
		m2["aaa"]["yyy"] = 2
		m2["aaa"]["zzz"] = 3
	}
	fmt.Println(m2)
}
```
>出力
```
map[string]map[string]int
map[]
----------------------
map[aaa:map[xxx:1 yyy:2 zzz:3]]
```
キー文字列を連結した文字列 をキーにすることで、多次元Mapを回避するといった代替手法も見かけます。
## ポインタ
```go
package main

import "fmt"

func main() {
	var p *int // intを指すポインタ型
	fmt.Printf("%T\n", p)
	fmt.Println(p)

	i := 100
	p = &i

	fmt.Println("-----------------------")
	fmt.Println(p)
	fmt.Println(*p)
	fmt.Println(i)
	fmt.Println(&i)

	*p = 200
	fmt.Println("-----------------------")
	fmt.Println(p)
	fmt.Println(*p)
	fmt.Println(i)
	fmt.Println(&i)

	i = 300
	fmt.Println("-----------------------")
	fmt.Println(p)
	fmt.Println(*p)
	fmt.Println(i)
	fmt.Println(&i)
}
```
>出力
```
*int
<nil>
-----------------------
0xc0000160a0
100
100
0xc0000160a0
-----------------------
0xc0000160a0
200
200
0xc0000160a0
-----------------------
0xc0000160a0
300
300
0xc0000160a0
```
## Channel チャネル
Channelは、goroutineで並行実行される関数とデータをやりとりするのに活用できます。
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// <-chan int: 受信専用
func receivePrint(name string, c <-chan int, wg *sync.WaitGroup) {
	for {
		time.Sleep(1000 * time.Millisecond)
		i, ok := <-c
		fmt.Printf("[Receive]\tname: %v\tcap: %v\tlen: %v\ti: %v\tok: %v\n", name, cap(c), len(c), i, ok)
		if ok == false {
			break
		}
	}
	fmt.Printf("[Done]\t\tname: %v\n", name)
	wg.Done()
}

func main() {
	var wg sync.WaitGroup

	// バッファーサイズ5のチャネル
	c := make(chan int, 5)
	fmt.Printf("%T\n", c)
	fmt.Println(c)

	wg.Add(2)
	go receivePrint("1st", c, &wg)
	go receivePrint("2nd", c, &wg)

	for i := 0; i < 10; i++ {
		c <- i
		fmt.Printf("[Send]\t\tname: main\tcap: %v\tlen: %v\n", cap(c), len(c))
	}
	close(c)
	wg.Wait()
	fmt.Println("[Done]\t\tname: main")
}
```
>出力
```bash
chan int
0xc000112000
[Send]          name: main      cap: 5  len: 1
[Send]          name: main      cap: 5  len: 2
[Send]          name: main      cap: 5  len: 3
[Send]          name: main      cap: 5  len: 4
[Send]          name: main      cap: 5  len: 5
[Send]          name: main      cap: 5  len: 4
[Send]          name: main      cap: 5  len: 5
[Receive]       name: 1st       cap: 5  len: 4  i: 1    ok: true
[Receive]       name: 2nd       cap: 5  len: 5  i: 0    ok: true
[Receive]       name: 2nd       cap: 5  len: 5  i: 2    ok: true
[Send]          name: main      cap: 5  len: 4
[Send]          name: main      cap: 5  len: 5
[Receive]       name: 1st       cap: 5  len: 4  i: 3    ok: true
[Receive]       name: 2nd       cap: 5  len: 5  i: 4    ok: true
[Receive]       name: 1st       cap: 5  len: 4  i: 5    ok: true
[Send]          name: main      cap: 5  len: 4
[Receive]       name: 1st       cap: 5  len: 3  i: 6    ok: true
[Receive]       name: 2nd       cap: 5  len: 2  i: 7    ok: true
[Receive]       name: 2nd       cap: 5  len: 1  i: 8    ok: true
[Receive]       name: 1st       cap: 5  len: 0  i: 9    ok: true
[Receive]       name: 2nd       cap: 5  len: 0  i: 0    ok: false
[Done]          name: 2nd
[Receive]       name: 1st       cap: 5  len: 0  i: 0    ok: false
[Done]          name: 1st
[Done]          name: main
```
## 関数
```go
package main

import "fmt"

func main() {
	f := func(x int) {
		fmt.Println("hello world '", x, "'.")
	}
	f(1)
	fmt.Printf("%T\n", f)
	fmt.Println(f)
}
```
>出力
```
hello world ' 1 '.
func(int)
0x10a8b80
```
# interface{}
interface{} はどんな型とも互換性のある型です。
```go
package main

import (
	"fmt"
)

func printInterface(i interface{})  {
	fmt.Printf("%v[%T]\n", i, i)

	switch v := i.(type) {
	case int:
		fmt.Printf("%v is int[%T]\n", v, v)
	case string:
		fmt.Printf("%v is string[%T]\n", v, v)
	default:
		fmt.Printf("default: %v[%T]\n", v, v)
	}
}

func main() {
	var i interface{}
	fmt.Printf("%v[%T]\n", i, i)

	fmt.Println("-----------------------------")
	i = "hello"
	fmt.Printf("%v[%T]\n", i, i)
	printInterface(i)

	fmt.Println("-----------------------------")
	i = 100
	fmt.Printf("%v[%T]\n", i, i)
	printInterface(i)

	fmt.Println("-----------------------------")
	i = true
	fmt.Printf("%v[%T]\n", i, i)
	printInterface(i)
}
```
>出力
```
<nil>[<nil>]
-----------------------------
hello[string]
hello[string]
hello is string[string]
-----------------------------
100[int]
100[int]
100 is int[int]
-----------------------------
true[bool]
true[bool]
default: true[bool]
```








