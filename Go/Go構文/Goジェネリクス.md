# Goジェネリクス
>[参考](https://www.wakuwakubank.com/posts/868-go-generics/)
より詳しくはこの記事
>[Go言語のジェネリクス入門(1)](https://zenn.dev/nobishii/articles/type_param_intro)

Genericsを利用すると「型が異なるだけで同じ処理をもつ複数の関数」を「1つの関数」として定義することができます

## Genericsを利用しない場合
```go
package main

import (
	"fmt"
)

func main() {
	stringValue := "111"
	intValue := 111

	fmt.Printf("sampleFuncString: %#v\n", sampleFuncString(stringValue))
	fmt.Printf("sampleFuncInt: %#v\n", sampleFuncInt(intValue))
}

func sampleFuncString(x string) string {
	return x
}

func sampleFuncInt(x int) int {
	return x
}
```
>実行結果
```
sampleFuncString: "111"
sampleFuncInt: 111
```
## Genericsの使い方
### comparableで型制約
- comparableは「比較可能な型」を表します
- comparableな型は「==」や「!=」で比較できる型です
```go
package main

import (
	"fmt"
)

func main() {
	stringValue := "111"
	intValue := 111
	boolValue := false

	fmt.Printf("sampleFuncGenerics1: %#v\n", sampleFuncGenerics1(stringValue))
	fmt.Printf("sampleFuncGenerics1: %#v\n", sampleFuncGenerics1(intValue))
	fmt.Printf("sampleFuncGenerics1: %#v\n", sampleFuncGenerics1(boolValue))
}

func sampleFuncGenerics1[T comparable](x T) T {
	return x
}
```
>実行結果
```
sampleFuncGenerics1: "111"
sampleFuncGenerics1: 111
sampleFuncGenerics1: false
```
sampleFuncGenerics1 の引数に指定できる値ですが、 ==, !=演算子で比較可能な値のみになります。
- スライスなどは ==演算子 で比較不可能なので利用できません。
### 型制約を複数指定する
```go
package main

import (
	"fmt"
)

func main() {
	stringValue := "111"
	intValue := 111

	fmt.Printf("sampleFuncGenerics2: %#v\n", sampleFuncGenerics2(stringValue))
	fmt.Printf("sampleFuncGenerics2: %#v\n", sampleFuncGenerics2(intValue))
}

func sampleFuncGenerics2[T string | int](x T) T {
	return x
}
```
>実行結果
```
sampleFuncGenerics2: "111"
sampleFuncGenerics2: 111
sampleFuncGenerics2: false
```
### 型制約をinterfaceで指定する
型パラメータを string または int だけに制約したい場合、以下のようにインタフェースを定義して実装することも可能です。
```go
package main

import (
	"fmt"
)

func main() {
	stringValue := "111"
	intValue := 111

	fmt.Printf("sampleFuncGenerics3: %#v\n", sampleFuncGenerics3(stringValue))
	fmt.Printf("sampleFuncGenerics3: %#v\n", sampleFuncGenerics3(intValue))
}

type SampleType interface {
	string | int
}

func sampleFuncGenerics3[T SampleType](x T) T {
	return x
}
```
>実行結果
```
sampleFuncGenerics3: "111"
sampleFuncGenerics3: 111
```
### Underlying Typeで型制約
以下例では [T ~int] といった形で ~(チルダ) を利用して型制約しています。
```go
package main

import (
	"fmt"
)

type SampleType1 int // underlying typeがint
type SampleType2 int // underlying typeがint

func main() {
	intValue1 := 111
	intValue2 := SampleType1(222)
	intValue3 := SampleType2(333)

	fmt.Printf("sampleFuncGenerics4: %#v\n", sampleFuncGenerics4(intValue1))
	fmt.Printf("sampleFuncGenerics4: %#v\n", sampleFuncGenerics4(intValue2))
	fmt.Printf("sampleFuncGenerics4: %#v\n", sampleFuncGenerics4(intValue3))
}

func sampleFuncGenerics4[T ~int](x T) T {
	return x
}
```
- `~`を利用することで、Underlying Type(基礎型) で型制約できます。

上記例では Underlying Typeがintの型のみ許可 しています。
































