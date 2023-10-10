# Go型変換
>[参考記事](https://www.wakuwakubank.com/posts/776-go-datatype/#index_id10)

## 型キャスト
```go
package main

import "fmt"

func main() {
	// int → float64
	var i1 int = 1
	var f1 float64 = float64(i1)
	// i1: 1(int) f1: 1(float64)
	fmt.Printf("i1: %v(%T) f1: %v(%T)\n", i1, i1, f1, f1)

	// float64 → int
	var f2 float64 = 1.11111
	var i2 int = int(f2)
	// i2: 1(int) f2: 1.11111(float64)
	fmt.Printf("i2: %v(%T) f2: %v(%T)\n", i2, i2, f2, f2)
}
```
## 型キャスト(ポインタ)
MyInt を int にキャストします。
```go
package main

import "fmt"

func main() {
	type MyInt int
	i1 := MyInt(1)

	var i2 *int
	i2 = (*int)(&i1)
	fmt.Printf("i1: %v(%T) &i1: %v(%T)\n", i1, i1, &i1, &i1)
	fmt.Printf("i2: %v(%T) *i2: %v(%T)\n", i2, i2, *i2, *i2)

	*i2 = 100
	fmt.Printf("i1: %v(%T) &i1: %v(%T)\n", i1, i1, &i1, &i1)
	fmt.Printf("i2: %v(%T) *i2: %v(%T)\n", i2, i2, *i2, *i2)
}
```
>出力
```
i1: 1(main.MyInt) &i1: 0xc0000160a0(*main.MyInt)
i2: 0xc0000160a0(*int) *i2: 1(int)
i1: 100(main.MyInt) &i1: 0xc0000160a0(*main.MyInt)
i2: 0xc0000160a0(*int) *i2: 100(int)
```
ポインタなので i2 を更新すると i1 の値も変わります。
## strconv.Atoi, strconv.ItoA
strconvを利用してstringとintを変換します。
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	// string → int
	s1 := "999"
	i1, _ := strconv.Atoi(s1)
	// s1: 999(string) i1: 999(int)
	fmt.Printf("s1: %v(%T) i1: %v(%T)\n", s1, s1, i1, i1)

	// int → string
	i2 := 999
	s2 := strconv.Itoa(i2)
	// s2: 999(string) i2: 999(int)
	fmt.Printf("s2: %v(%T) i2: %v(%T)\n", s2, s2, i2, i2)
}
```







































