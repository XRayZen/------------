# errors(New, Is, As), fmt.Errorf, panicとrecover
>[参考記事](https://www.wakuwakubank.com/posts/783-go-error-panic/)

Goには例外処理が存在しなく、関数の戻り値としてerrorインタフェースを返すことでエラーハンドリングするというルールが存在します。

エラーの定義、判定には、errorsパッケージの関数を活用できます。

また、panic、recoverといった組み込み関数が用意されており、ランタイムエラーの制御に利用できます。

Table of Contents
=================
- [errors(New, Is, As), fmt.Errorf, panicとrecover](#errorsnew-is-as-fmterrorf-panicとrecover)
- [Table of Contents](#table-of-contents)
- [エラーを返す関数](#エラーを返す関数)
- [エラーの定義](#エラーの定義)
  - [errors.New()とfmt.Errorf()](#errorsnewとfmterrorf)
  - [fmt.Errorf()でwrapErrorになるケース](#fmterrorfでwraperrorになるケース)
- [カスタムエラー](#カスタムエラー)
- [Errorの判別](#errorの判別)
  - [`errors.Is()`](#errorsis)
  - [`errors.As()`](#errorsas)
- [panicとrecover](#panicとrecover)
  - [panic](#panic)
  - [recover](#recover)
  - [goroutineのrecover](#goroutineのrecover)

# エラーを返す関数
以下のように、多くの関数はエラー値を返すように実装されています。

関数を呼び出す側は、エラーがnilかどうか判定して、nil以外であればエラー処理を行います。
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("--------------------------------")
	t1, err := time.Parse(time.RFC3339, "2020-12-01T00:00:00+09:00")
	if err != nil {
		fmt.Printf("type: %T\n", err)
		fmt.Printf("value: %v\n", err)
		return
	}
	fmt.Println(t1)

	fmt.Println("--------------------------------")
	t2, err := time.Parse(time.RFC3339, "ABC")
	if err != nil {
		fmt.Printf("type: %T\n", err)
		fmt.Printf("value: %v\n", err)
		return
	}
	fmt.Println(t2)
}
```
>Output
```
--------------------------------
2020-12-01 00:00:00 +0900 JST
--------------------------------
type: *time.ParseError
value: parsing time "ABC" as "2006-01-02T15:04:05Z07:00": cannot parse "ABC" as "2006"
```
# エラーの定義
## errors.New()とfmt.Errorf()
errors.New() や fmt.Errorf() を利用してエラーを定義できます。
```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err1 := errors.New("test error1")
	fmt.Printf("type: %T\n", err1)
	fmt.Printf("value: %v\n", err1)

	err2 := fmt.Errorf("test error2 %v", "wakuwaku bank")
	fmt.Printf("type: %T\n", err2)
	fmt.Printf("value: %v\n", err2)
}
```
>Output
```
type: *errors.errorString
value: test error1
type: *errors.errorString
value: test error2 wakuwaku bank
```
## fmt.Errorf()でwrapErrorになるケース
fmt.Errorf で %w をフォーマット指定子に指定すると、wrapError を返します。
```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err1 := errors.New("test error1")
	fmt.Printf("type: %T\n", err1)
	fmt.Printf("value: %v\n", err1)

	wrapErr := fmt.Errorf("test error2 %w", err1)
	fmt.Printf("type: %T\n", wrapErr)
	fmt.Printf("value: %v\n", wrapErr)

	unwrapErr := errors.Unwrap(wrapErr)
	fmt.Printf("type: %T\n", unwrapErr)
	fmt.Printf("value: %v\n", unwrapErr)
}
```
>Output
```
type: *errors.errorString
value: test error1
type: *fmt.wrapError
value: test error2 test error1
type: *errors.errorString
value: test error1
```
Unwrapメソッド で、ラップ元のエラーを取得できます。wrapErrorは後述のエラー判定で活用されます。
# カスタムエラー
独自のエラータイプを作りたい場合、組み込みインターフェースである error interface を実装します。
```go
type error interface {
    Error() string
}
```
以下、実装例です。SampleErrorという独自のエラータイプを定義しています。
error interface を実装するため、Errorメソッドを実装しています。
```go
type SampleError struct {
	Field1 string
	Field2 string
}

func (e *SampleError) Error() string {
	return fmt.Sprintf("[sample error] Field1: %v Field2: %v", e.Field1, e.Field2)
}

func sampleFunc(i interface{}) (int, error) {
	switch v := i.(type) {
	case int:
		return v * 10, nil
	default:
		return 0, &SampleError{"hello", "world"}
	}

}

func main() {
	v1, err := sampleFunc(5)
	if err != nil {
		fmt.Printf("type: %T\n", err)
		fmt.Printf("value: %v\n", err)
		return
	}
	fmt.Println(v1)

	v2, err := sampleFunc("hello")
	if err != nil {
		fmt.Printf("type: %T\n", err)
		fmt.Printf("value: %v\n", err)
		return
	}
	fmt.Println(v2)
}
```
>Output
```
50
type: *main.SampleError
value: [sample error] Field1: hello Field2: world
```
# Errorの判別
## `errors.Is()`
`errors.Is()` は、「特定のエラーであるのか」「特定のエラーをWrapしたエラーであるのか」を判別するのに活用できます。
```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err1 := errors.New("aaa")
	err2 := errors.New("bbb")
	err3 := errors.New("ccc")
	err4 := errors.New("aaa")
	wrappedErr1 := fmt.Errorf("wrapped %w", err1)
	wrappedWrappedErr1 := fmt.Errorf("wrapped %w", wrappedErr1)

	fmt.Printf("errors.Is(err1, err1) = %v\n", errors.Is(err1, err1))
	fmt.Printf("errors.Is(err1, err2) = %v\n", errors.Is(err1, err2))
	fmt.Printf("errors.Is(err1, err3) = %v\n", errors.Is(err1, err3))
	fmt.Printf("errors.Is(err1, err4) = %v\n", errors.Is(err1, err4))
	fmt.Printf("errors.Is(err1, wrappedErr1) = %v\n", errors.Is(err1, wrappedErr1))
	fmt.Printf("errors.Is(wrappedErr1, err1) = %v\n", errors.Is(wrappedErr1, err1))
	fmt.Printf("errors.Is(wrappedWrappedErr1, err1) = %v\n", errors.Is(wrappedWrappedErr1, err1))
	fmt.Printf("errors.Is(wrappedWrappedErr1, wrappedErr1) = %v\n", errors.Is(wrappedWrappedErr1, wrappedErr1))
}
```
>Output
```
errors.Is(err1, err1) = true
errors.Is(err1, err2) = false
errors.Is(err1, err3) = false
errors.Is(err1, err4) = false
errors.Is(err1, wrappedErr1) = false
errors.Is(wrappedErr1, err1) = true
errors.Is(wrappedWrappedErr1, err1) = true
errors.Is(wrappedWrappedErr1, wrappedErr1) = true
```
## `errors.As()`
`errors.As() `は、型レベルで一致するか判別するのに活用できます。
```go
package main

import (
	"errors"
	"fmt"
)

type MyError1Interface interface {
	Error() string
	Echo() string
}

type MyError1 struct{}

func (e *MyError1) Error() string {
	return "this is my error1"
}

func (e *MyError1) Echo() string {
	return "hello world"
}

type MyError2 struct{}

func (e *MyError2) Error() string {
	return "this is my error2"
}

func main() {
	var myError1Interface MyError1Interface // MyError1Interface interface

	err11 := &MyError1{}
	err12 := &MyError1{}
	err21 := &MyError2{}
	err22 := &MyError2{}

	wrappedErr11 := fmt.Errorf("wrapped %w", err11)

	fmt.Printf("errors.As(err11, &myError1Interface) = %v\n", errors.As(err11, &myError1Interface))
	fmt.Printf("errors.As(err11, &err11) = %v\n", errors.As(err11, &err11))
	fmt.Printf("errors.As(err11, &err12) = %v\n", errors.As(err11, &err12))
	fmt.Printf("errors.As(err11, &err21) = %v\n", errors.As(err11, &err21))
	fmt.Printf("errors.As(err11, &err22) = %v\n", errors.As(err11, &err22))

	fmt.Println("------------------------------------------")

	fmt.Printf("errors.As(err21, &myError1Interface) = %v\n", errors.As(err21, &myError1Interface))
	fmt.Printf("errors.As(err21, &err11) = %v\n", errors.As(err21, &err11))
	fmt.Printf("errors.As(err21, &err12) = %v\n", errors.As(err21, &err12))
	fmt.Printf("errors.As(err21, &err21) = %v\n", errors.As(err21, &err21))
	fmt.Printf("errors.As(err21, &err22) = %v\n", errors.As(err21, &err22))

	fmt.Println("------------------------------------------")

	fmt.Printf("errors.As(wrappedErr11, &myError1Interface) = %v\n", errors.As(wrappedErr11, &myError1Interface))
	fmt.Printf("errors.As(wrappedErr11, &err11) = %v\n", errors.As(wrappedErr11, &err11))
	fmt.Printf("errors.As(wrappedErr11, &err12) = %v\n", errors.As(wrappedErr11, &err12))
	fmt.Printf("errors.As(wrappedErr11, &err21) = %v\n", errors.As(wrappedErr11, &err21))
	fmt.Printf("errors.As(wrappedErr11, &err22) = %v\n", errors.As(wrappedErr11, &err22))
}
```
>Output
```
errors.As(err11, &myError1Interface) = true
errors.As(err11, &err11) = true
errors.As(err11, &err12) = true
errors.As(err11, &err21) = false
errors.As(err11, &err22) = false
------------------------------------------
errors.As(err21, &myError1Interface) = false
errors.As(err21, &err11) = false
errors.As(err21, &err12) = false
errors.As(err21, &err21) = true
errors.As(err21, &err22) = true
------------------------------------------
errors.As(wrappedErr11, &myError1Interface) = true
errors.As(wrappedErr11, &err11) = true
errors.As(wrappedErr11, &err12) = true
errors.As(wrappedErr11, &err21) = false
errors.As(wrappedErr11, &err22) = false
```
# panicとrecover
## panic
配列のインデックス範囲外にアクセスするといった処理があると、panicと呼ばれる状態になり処理が終了します。
```go
package main

import (
	"fmt"
)

func sampleFunc1() {
	fmt.Println("wakuwaku")
	a := []int{1, 2, 3}
	fmt.Println(a[3])
	fmt.Println("bank")
}

func main() {
	fmt.Println("hello")
	sampleFunc1()
	fmt.Println("world")
}
```
>Output
```
hello
wakuwaku
panic: runtime error: index out of range [3] with length 3

goroutine 1 [running]:
main.sampleFunc1()
        /tmp/sample1.go:10 +0x85
main.main()
        /tmp/sample1.go:16 +0x7e
exit status 2
```
組み込み関数panic を利用して、明示的にpanicを発生させることもできます。
```go
package main

import (
	"fmt"
)

func sampleFunc2() {
	fmt.Println("wakuwaku")
	panic("panic test")
	fmt.Println("bank")
}

func main() {
	fmt.Println("hello")
	sampleFunc2()
	fmt.Println("world")
}
```
>Output
```
hello
wakuwaku
panic: panic test

goroutine 1 [running]:
main.sampleFunc2()
        /tmp/sample2.go:7 +0x95
main.main()
        /tmp/sample2.go:13 +0x7e
exit status 2
```
## recover
panic発生後も処理を継続させたい場合、recover関数を利用します。
- 以下例では、deferで登録した関数内でrecoverを実行して、処理を継続させています。
```go
package main

import (
	"fmt"
)

func sampleFunc3() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("recover: ", r)
		}
	}()

	fmt.Println("wakuwaku")
	a := []int{1, 2, 3}
	fmt.Println(a[3])
	fmt.Println("bank")
}

func main() {
	fmt.Println("hello")
	sampleFunc3()
	fmt.Println("world")
}
```
>Output
```
hello
wakuwaku
recover:  runtime error: index out of range [3] with length 3
world
```
## goroutineのrecover
まずは、NGケースを確認します。
- 以下例では、goroutineで実行する関数内でpanicが発生して、処理が途中終了してます。
```go
package main

import (
	"fmt"
	"sync"
)

func echo1(i int, wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Printf("wakuwaku %v\n", i)
	a := []int{1, 2, 3}
	fmt.Println(a[3]) // panic
	fmt.Printf("bank %v\n", i)
}

func sampleFunc4() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("recover: ", r)
		}
	}()

	var wg sync.WaitGroup
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go echo1(i, &wg)
	}
	wg.Wait()
}

func main() {
	fmt.Println("hello")
	sampleFunc4()
	fmt.Println("world")
}
```
>Output
```
hello
wakuwaku 2
wakuwaku 0
panic: runtime error: index out of range [3] with length 3

goroutine 20 [running]:
main.echo1(0x2, 0x1400012a010)
        /tmp/sample.go:12 +0xbc
created by main.sampleFunc4
        /tmp/sample.go:26 +0x94
```



























