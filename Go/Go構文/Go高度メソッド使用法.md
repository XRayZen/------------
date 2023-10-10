# Goメソッドの一歩進んだ使い方
>https://www.wakuwakubank.com/posts/778-go-func/

## 高階関数 ( 関数に関数を渡す )
以下、高階関数(他の関数を引数として受け取れる関数)の実装例です。
```go
package main

import "fmt"

func add(x, y int) int {
	return x + y
}

func sub(x, y int) int {
	return x - y
}

func ope(
	x, y int,
	cb func(int, int) int,
) int {
	return cb(x, y) + 10
}

func main() {
	fmt.Println(ope(10, 5, add))
	fmt.Println(ope(10, 5, sub))
}
```
>出力
```
25
15
```
## 再帰関数
再帰関数で階乗を求めてみます。
```go
package main

import "fmt"

func factorial(i int) int {
	if i < 0 {
		return -1
	} else if i == 0 {
		return 1
	} else {
		return i * factorial(i-1)
	}
}

func main() {
	fmt.Println(factorial(-10))
	fmt.Println(factorial(0))
	fmt.Println(factorial(4))
}
```
>出力
```
-1
1
24
```
## 関数を要素に持つスライス
```go
package main

import "fmt"

func main() {
	var tasks = []func(int) string{
		func(i int) string { return fmt.Sprintf("aaa %d", i) },
		func(i int) string { return fmt.Sprintf("bbb %d", i) },
		func(i int) string { return fmt.Sprintf("ccc %d", i) },
	}

	for i, task := range tasks {
		fmt.Println(task(i))
	}
}
```
>出力
```
aaa 0
bbb 1
ccc 2
```
## 即時関数
```go
package main

import "fmt"

func main() {
	func(m string) {
		fmt.Println(m)
	}("Hello World")
}
```
>出力
```
Hello World
```





























