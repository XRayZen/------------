# 処理フロー(if, for, switch, defer, select)
>https://www.wakuwakubank.com/posts/777-go-basic-control-flow/

メニュー
- [処理フロー(if, for, switch, defer, select)](#処理フローif-for-switch-defer-select)
  - [if](#if)
  - [for](#for)
    - [whileのように利用](#whileのように利用)
  - [range｜配列, スライス](#range配列-スライス)
    - [range｜map](#rangemap)
    - [range｜文字列](#range文字列)
  - [break](#break)
    - [break label](#break-label)
  - [continue](#continue)
    - [continue label](#continue-label)
  - [switch](#switch)
    - [利用例1](#利用例1)
    - [利用例2](#利用例2)
    - [利用例3](#利用例3)
  - [defer](#defer)
  - [select](#select)
  - [スコープ](#スコープ)
  - [暗黙的なブロック](#暗黙的なブロック)

## if
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	i := 5
	if i > 0 {
		fmt.Println("xxx")
	}

	if t := time.Now(); t.Hour() < 14 {
		// 変数tのスコープはifの終わりまで
		fmt.Println("Hour: ", t.Hour())
	}
}
```
>出力
```
xxx
Hour:  12
```
## for
基本
```go
package main

import "fmt"

func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}
```
>出力
```
45
```
### whileのように利用
```go
package main

import "fmt"

func main() {
	sum := 1
	for sum < 20 {
		sum += sum
		fmt.Println(sum)
	}
	fmt.Println(sum)
}
```
>>出力
```
2
4
8
16
32
32
```
## range｜配列, スライス
```go
package main

import "fmt"

func main() {
	s := []string{"aaa", "bbb", "ccc"}

	for i, v := range s {
		fmt.Printf("index: %d  value: %v\n", i, v)
	}
}
```
>>出力
```
index: 0  value: aaa
index: 1  value: bbb
index: 2  value: ccc
```
### range｜map
```go
package main

import "fmt"

func main() {
	m := map[string]int{
		"aaa": 100,
		"bbb": 200,
		"ccc": 300,
	}

	for k, v := range m {
		fmt.Printf("key: %v  value: %v\n", k, v)
	}
}
```
>出力
```
key: aaa  value: 100
key: bbb  value: 200
key: ccc  value: 300
```
### range｜文字列
```go
package main

import "fmt"

func main() {
	s := "わくわくBank"

	for i, v := range s {
		fmt.Printf("%T i:%v v:%v\n", v, i, v)
	}
}
```
>出力
```
int32 i:0 v:12431
int32 i:3 v:12367
int32 i:6 v:12431
int32 i:9 v:12367
int32 i:12 v:66
int32 i:13 v:97
int32 i:14 v:110
int32 i:15 v:107
```
## break
```go
package main

import "fmt"

func main() {
	for i := 0; i < 3; i++ {
		fmt.Printf("i: %d\n", i)

		for j := 0; j < 3; j++ {
			fmt.Printf("j: %d\n", j)
			if j == 2 {
				fmt.Println("inner loop")
				break
			}
		}

		if i == 2 {
			fmt.Println("outer loop")
			break
		}
	}
	fmt.Println("done")
}
```
>出力
```
i: 0
j: 0
j: 1
j: 2
inner loop
i: 1
j: 0
j: 1
j: 2
inner loop
i: 2
j: 0
j: 1
j: 2
inner loop
outer loop
done
```
### break label
```go
package main

import "fmt"

func main() {
OuterLoop:
	for i := 0; i < 3; i++ {
		fmt.Printf("i: %d\n", i)

		for j := 0; j < 3; j++ {
			fmt.Printf("j: %d\n", j)
			if j == 2 {
				fmt.Println("inner loop")
				break OuterLoop
			}
		}

		if i == 2 {
			fmt.Println("outer loop")
			break
		}
	}
	fmt.Println("done")
}
```
>出力
```
i: 0
j: 0
j: 1
j: 2
inner loop
done
```
## continue
```go
package main

import "fmt"

func main() {
	for i := 0; i < 3; i++ {
		fmt.Printf("i: %d\n", i)

		for j := 0; j < 3; j++ {
			fmt.Printf("j: %d\n", j)
			if j == 2 {
				fmt.Println("inner loop")
				continue
			}
		}

		if i == 2 {
			fmt.Println("outer loop")
			continue
		}
	}
	fmt.Println("done")
}
```
>出力
```
i: 0
j: 0
j: 1
j: 2
inner loop
i: 1
j: 0
j: 1
j: 2
inner loop
i: 2
j: 0
j: 1
j: 2
inner loop
outer loop
done
```
### continue label
```go
package main

import "fmt"

func main() {
OuterLoop:
	for i := 0; i < 3; i++ {
		fmt.Printf("i: %d\n", i)

		for j := 0; j < 3; j++ {
			fmt.Printf("j: %d\n", j)
			if j == 2 {
				fmt.Println("inner loop")
				continue OuterLoop
			}
		}

		if i == 2 {
			fmt.Println("outer loop")
			continue
		}
	}
	fmt.Println("done")
}
```
>出力
```
i: 0
j: 0
j: 1
j: 2
inner loop
i: 1
j: 0
j: 1
j: 2
inner loop
i: 2
j: 0
j: 1
j: 2
inner loop
done
```
## switch
### 利用例1
break は不要です。
```go
package main

import (
	"fmt"
)

func main() {
	s := "bbb"
	switch s {
	case "aaa":
		fmt.Println("case aaa")
	case "bbb":
		fmt.Println("case bbb")
	default:
		fmt.Println("case default")
	}
}
```
>出力
```
case bbb
```
### 利用例2
```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Println(runtime.NumCPU())

	switch {
	case runtime.NumCPU() < 8:
		fmt.Println("case runtime.NumCPU() < 8")
	case runtime.NumCPU() < 16:
		fmt.Println("case runtime.NumCPU() < 16")
	case runtime.NumCPU() < 32:
		fmt.Println("case runtime.NumCPU() < 32")
	case runtime.NumCPU() < 32:
		fmt.Println("case runtime.NumCPU() < 64")
	default:
		fmt.Println("case default")
	}
}
```
>出力
```
16
case runtime.NumCPU() < 32
```
### 利用例3
```go
package main

import (
	"fmt"
)

func main() {
	var i interface{} = "hello"

	switch v := i.(type) {
	case int:
		fmt.Printf("%v is int[%T]\n", v, v)
	case string:
		fmt.Printf("%v is string[%T]\n", v, v)
	default:
		fmt.Printf("default: %v[%T]\n", v, v)
	}
}
```
>出力
```
hello is string[string]
```
## defer
deferで指定した処理は、関数の処理が終わったあとに実行されます。
```go
package main

import "fmt"

func main() {
	defer fmt.Println("world1")
	defer fmt.Println("world2")

	fmt.Println("hello")
}
```
>出力
```
hello
world2
world1
```
メソッドチェーンで利用するときは注意が必要です。以下例です。
```go
package main

import "fmt"

type deferTest int

func (t *deferTest) func1() *deferTest {
	fmt.Println("func1")
	return t
}

func (t *deferTest) func2() *deferTest {
	fmt.Println("func2")
	return t
}

func (t *deferTest) func3() *deferTest {
	fmt.Println("func3")
	return t
}

func main() {
	var dt deferTest
	defer dt.func1().func2().func3()

	fmt.Println("main func")
}
```
>出力
```
func1
func2
main func
func3
```
## select
並行処理で複数チャネルを処理するときに、select という制御構文を活用できます。
```go
package main

import (
	"fmt"
	"time"
)

func test1(ch chan<- string) {
	for {
		ch <- "test1"
		time.Sleep(2 * time.Second)
	}
}

func test2(ch chan<- string) {
	for {
		ch <- "test2"
		time.Sleep(4 * time.Second)
	}
}

func test3(quit chan<- int) {
	time.Sleep(10 * time.Second)
	quit <- 0
}

func main() {
	c1 := make(chan string)
	c2 := make(chan string)
	quit := make(chan int)
	go test1(c1)
	go test2(c2)
	go test3(quit)

	cnt := 0
	for {
		select {
		case s1 := <-c1:
			fmt.Println(s1)
		case s2 := <-c2:
			fmt.Println(s2)
		case <-quit:
			fmt.Println("quit")
			return
		default:
			cnt = cnt + 1
			fmt.Printf("(cnt: %v)\n", cnt)
			time.Sleep(1 * time.Second)
		}
	}
}
```
>出力
```
(cnt: 1)
test1
test2
(cnt: 2)
(cnt: 3)
(cnt: 4)
test1
(cnt: 5)
test2
(cnt: 6)
(cnt: 7)
test1
(cnt: 8)
(cnt: 9)
test1
test2
(cnt: 10)
quit
```
## スコープ
明示的なブロック
{ } (中括弧) で囲むことでブロック(Block)を宣言でき、スコープを分離できます。
```go
package main

import (
	"fmt"
)

func main() {
	x := 1
	y := 2

	// Blockを宣言
	{
		x = 10
		y := 20
		z := 30
		fmt.Printf("x = %v\n", x)
		fmt.Printf("y = %v\n", y)
		fmt.Printf("z = %v\n", z)
	}

	// Block内で上書きされたので 10 になる
	fmt.Printf("x = %v\n", x)

	// Block内で定義されたyとは異なるので 2 のまま
	fmt.Printf("y = %v\n", y)

	// zはBlock内でのみ定義されているので、ここでは利用不可能
	//fmt.Printf("z = %v\n", z)
}
```
>出力
```
x = 10
y = 20
z = 30
x = 10
y = 2
```
## 暗黙的なブロック
「if for switch の宣言」「 switch select の各句」は、暗黙的にブロックと見なされて、スコープを分離します。










