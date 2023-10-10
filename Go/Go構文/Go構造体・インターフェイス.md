# 構造体
>[struct, method, interfaceの活用](https://www.wakuwakubank.com/posts/779-go-struct-method-interface/)

Goの特徴として「クラス構文が存在しない」「継承が存在しない」などが挙げられます。
代わりに、構造体に処理(メソッド)を紐づけることができます。

Table of Contents
=================
- [構造体](#構造体)
- [Table of Contents](#table-of-contents)
- [struct｜構造体](#struct構造体)
	- [typeで型定義](#typeで型定義)
	- [構造体は値型であり関数呼び出し時に考慮する必要](#構造体は値型であり関数呼び出し時に考慮する必要)
	- [コンストラクタ](#コンストラクタ)
	- [フィールドにタグ付け](#フィールドにタグ付け)
- [(構造体に紐付ける)method](#構造体に紐付けるmethod)
	- [public, private](#public-private)
	- [値レシーバ、ポインタレシーバ](#値レシーバポインタレシーバ)
	- [処理の再利用( 埋め込み型 )](#処理の再利用-埋め込み型-)
- [interface](#interface)
- [構造体の初期化](#構造体の初期化)
	- [構造体の初期化（型省略）](#構造体の初期化型省略)
	- [構造体のフィールドを代入](#構造体のフィールドを代入)

# struct｜構造体
## typeで型定義
type で新しく T型 を定義しています。
- type とは、既にある型（型リテラル）に対して新しい名前を与えるための機能
```go
type T struct {
	PublicField  int // 先頭大文字はpublic(外部packageから参照可能)
	privateField int // 先頭小文字はprivate(外部packageから参照不可)
}

func main() {
	t := T{PublicField: 100, privateField: 200}
	fmt.Printf("%+v\n", t)
	// Output
	// {PublicField:100 privateField:200}
}
```
構造体のフィールドですが、頭文字を 大文字 にするか 小文字 にするかで、外部packageからのアクセス権が変わります。
## 構造体は値型であり関数呼び出し時に考慮する必要
構造体は参照型ではなく値型です。
- そのため、関数を呼び出すとき値のコピーが行われて、元の構造体は影響を受けません。

もし、関数で元の構造体を更新したい場合、以下 func2 のようにポインタを受け取るように実装する必要があります。
```go
type T1 struct {
	Field1 int
	Field2 int
}

func func1(t T1) {
	t.Field1 = t.Field1 * 2
	t.Field2 = t.Field2 * 2
}

func func2(t *T1) {
	t.Field1 = t.Field1 * 2
	t.Field2 = t.Field2 * 2
}

func main() {
	t1 := T1{Field1: 100, Field2: 200}
	fmt.Printf("%+v\n", t1)
	// Output
	// {Field1:100 Field2:200}
	fmt.Println("--------------")
	func1(t1)
	fmt.Printf("%+v\n", t1)
	// Output 構造体のフィールド値が変わらない
	// {Field1:100 Field2:200}
	fmt.Println("--------------")
	func2(&t1)
	fmt.Printf("%+v\n", t1)
	// Output ポインタで構造体のフィールド値が変わる
	// {Field1:200 Field2:400}
}
```
## コンストラクタ
コンストラクタとは、構造体を生成する関数のことです。
goにコンストラクタは存在しません。
- 代わりに、New や NewXxx のような関数名でコンストラクタのような処理を実装する命名上の慣習があります。
```go
type T4 struct {
	Field1 int
	Field2 int
}

func NewT4(x, y int) *T4 {
	return &T4{Field1: x, Field2: y}
}

func main() {
	t4 := NewT4(10, 20)
	fmt.Printf("%+v\n", t4)
	fmt.Println(t4.Field1)
	fmt.Println(t4.Field2)
}
```
>Output
```
&{Field1:10 Field2:20}
10
20
```
## フィールドにタグ付け
構造体のフィールドには、タグ付けできます。
```go
package main

import (
	"fmt"
	"reflect"
)

type T2 struct {
	Field1 int "aaa"
	Field2 int "bbb"
}

func main() {
	t2 := T2{Field1: 100, Field2: 200}

	t := reflect.TypeOf(t2)
	fmt.Println(t.Field(0))
	fmt.Println(t.Field(0).Tag)
	fmt.Println(t.Field(1))
	fmt.Println(t.Field(1).Tag)
}
```
>Output
```
{Field1  int aaa 0 [0] false}
aaa
{Field2  int bbb 8 [1] false}
bbb
```
- GoのORMであるGORMでは、タグ付けを利用して、構造体のフィールド名とDBのカラム名を紐づけています。
# (構造体に紐付ける)method
Goにはクラス構文が存在しません。
代わりに、typeで定義した(構造体)型に処理(method)を紐づけることができます。
## public, private
method名の頭文字を 大文字 にするか 小文字 にするかで、外部packageからのアクセス権が変わります。
- 下記例では、Method1 は外部packageから利用できますが、method2 は外部packageから利用できません。
```go
package main

import "fmt"

type T3 struct {
	Field1 int
	Field2 int
}

func (v T3) Method1() {
	fmt.Println(v.Field1)
}

func (v T3) method2() {
	fmt.Println(v.Field2)
}

func main() {
	t3 := T3{Field1: 100, Field2: 200}
	t3.Method1()
	t3.method2()
}
```
>Output
```
100
200
```
## 値レシーバ、ポインタレシーバ
以下のように 値レシーバ だとstructの内容を更新できません。
- structの内容を更新したい場合、ポインタレシーバ を定義します。
```go
package main

import (
	"errors"
	"fmt"
)

type T5 struct {
	Field1 int
	Field2 int
}

// Method1 値レシーバ
func (v T5) Method1(x int) {
	v.Field1 = x
}

// Method2 ポインタレシーバ
func (v *T5) Method2(x int) error {
	if v == nil {
		return errors.New("nil error")
	}

	v.Field1 = x
	return nil
}

func main() {
	t5 := T5{Field1: 100, Field2: 200}
	fmt.Printf("%+v\n", t5)

	t5.Method1(10000)
	fmt.Printf("%+v\n", t5)

	if err := t5.Method2(10000); err != nil {
		fmt.Println(err)
	}
	fmt.Printf("%+v\n", t5)

	var t5nil *T5
	t5nil = nil
	if err := t5nil.Method2(10000); err != nil {
		fmt.Println(err)
	}
	fmt.Printf("%+v\n", t5nil)
}
```
>Output
```
{Field1:100 Field2:200}
{Field1:100 Field2:200}
{Field1:10000 Field2:200}
nil error
<nil>
```
ポインタレシーバではnil の可能性があるため nil のチェックを行っています。もし nil のチェックを外した場合、nilの操作をする時点でpanicになります。
## 処理の再利用( 埋め込み型 )
Goには、継承はありません。
- 処理を再利用したいときなどは、埋め込み型を活用できます。

下記例では T6型 に T6Base型 を埋め込んでいます。
```go
type T6Base struct {
	Field1 int
	Field2 int
}

func (v T6Base) Method1() {
	fmt.Println("T6Base\tMethod1 Field1:", v.Field1, " Field2:", v.Field2)
}

type T6 struct {
	T6Base
	Field3 int
}

func (v T6) Method2() {
	fmt.Println("T6\tMethod2 Field1:", v.Field1, " Field2:", v.Field2, " Field3:", v.Field3)
}

func NewT6(x, y, z int) *T6 {
	return &T6{T6Base{x, y}, z}
}

func main() {
	t6 := NewT6(100, 200, 300)
	fmt.Printf("%+v\n", t6)
	fmt.Printf("%+v\n", t6.Field1)
	fmt.Printf("%+v\n", t6.Field2)
	fmt.Printf("%+v\n", t6.Field3)

	t6.Method1()
	t6.Method2()
}
```
>Output
```
&{T6Base:{Field1:100 Field2:200} Field3:300}
100
200
300
T6Base  Method1 Field1: 100  Field2: 200
T6      Method2 Field1: 100  Field2: 200  Field3: 300
```
ただし、埋め込み型を利用すると、以下のようなデメリットもあります。
- T6がどういったメソッドを持っているか分かりづらい。
- T6BaseでMethod1が削除されたときに破壊的変更が発生する。

面倒ですが、基本的には以下のようにWrapする形で同じメソッドを再定義したほうが良い
```go
type T6 struct {
	base   *T6Base
	Field3 int
}

func (v T6) Method1() {
	v.base.Method1()
}

func (v T6) Method2() {
	fmt.Println("T6\tMethod2 Field1:", v.base.Field1, " Field2:", v.base.Field2, " Field3:", v.Field3)
}

func NewT6(x, y, z int) *T6 {
	return &T6{base: &T6Base{x, y}, Field3: z}
}
```
# interface
interfaceでは、メソッド名だけを宣言します。
- Goでは該当interfaceで宣言した同一メソッドを実装することで、該当interfaceを実装できたとみなされる
- implementsなどの宣言は不要です。
```go
package main

import (
	"fmt"
)

type MyInterface interface {
	MethodAaa(x, y int) string
}

type MyS1 struct {
	Name string
}

func (s MyS1) MethodAaa(x, y int) string {
	sum := x + y
	return fmt.Sprintf("MyS1 Name: %+v sum: %v\n", s.Name, sum)
}

type MyS2 struct {
	Name string
}

func (s MyS2) MethodAaa(x, y int) string {
	sub := x - y
	return fmt.Sprintf("MyS2 Name: %+v sub: %v\n", s.Name, sub)
}

func main() {
	f := func(i MyInterface) {
		fmt.Print(i.MethodAaa(100, 50))
	}
	myS1 := MyS1{"hello"}
	myS2 := MyS2{"world"}

	f(myS1)
	f(myS2)
}
```
>Output
```
MyS1 Name: hello sum: 150
MyS2 Name: world sub: 50
```













# 構造体の初期化
```go
person := struct {
	name       string
	age        int
	gender     string
	weight     int
}{
	name:       "Tanaka Hanako",
	age:        25,
	gender:     "female",
	weight:     50,
}

fmt.Printf("%#v", person) // => struct { name string; age int; gender string; weight int }{name:"Tanaka Hanako", age:25, gender:"female", weight:50}
```
## 構造体の初期化（型省略）
構造体の定義でtype を使用した場合、定義した順番どおりに値を設定するのであれば、フィールド名を省略して書くこともできます
```go
構造体名 := 構造体名{値1, 値2, 値3}
```
## 構造体のフィールドを代入
```go
構造体変数名.フィールド名 = 値
```
>構造体のフィールドを参照
```go
構造体変数名.フィールド名
```
>例
```go
person.name = "Tanaka Hanako"
fmt.Printf("%#v", person.name) // => "Tanaka Hanako"
```