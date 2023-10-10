# 文字列操作と関連パッケージ(fmt, strings, strconv)
>[参考記事](https://www.wakuwakubank.com/posts/780-go-string/)

Goの文字列操作について確認します。
マルチバイト文字列を扱うときの注意点や、文字列操作で役立つパッケージ(fmt, strings, strconv, regexp)について取り上げます。

Table of Contents
=================
- [文字列操作と関連パッケージ(fmt, strings, strconv)](#文字列操作と関連パッケージfmt-strings-strconv)
- [Table of Contents](#table-of-contents)
- [文字列操作の基礎](#文字列操作の基礎)
  - [string型とrune型](#string型とrune型)
  - [文字列長を取得](#文字列長を取得)
  - [文字へのアクセス](#文字へのアクセス)
  - [エスケープシーケンス](#エスケープシーケンス)
  - [文字列連結](#文字列連結)
- [fmt package](#fmt-package)
  - [Print, Sprint](#print-sprint)
  - [Println, Sprintln](#println-sprintln)
  - [Printf, Sprintf](#printf-sprintf)
  - [フォーマット文字列(verb)](#フォーマット文字列verb)
- [strings package](#strings-package)
- [strconv package](#strconv-package)

# 文字列操作の基礎
## string型とrune型
ダブルクォーテーションで囲むとstring型になります。シングルクオートだとrune型になります。
```go
package main

import (
	"fmt"
)

func main() {
	var (
		s1 string = "abcdefg"
		s2 string = "あいうえお"

		s3 rune = 'a'
		s4 rune = 'あ'
	)

	fmt.Println(s1, s2, s3, s4, string(s3), string(s4))
}
```
>出力
```
abcdefg あいうえお 97 12354 a あ
```
## 文字列長を取得
マルチバイト文字の場合、rune型にしてからlen関数で取得します。
```go
package main

import (
	"fmt"
)

func main() {
	var (
		// 文字列はダブルクォーテーション
		s1 string = "abcdefg"
		s2 string = "あいうえお"
	)

	fmt.Println(len(s1), len([]rune(s1)))
	fmt.Println(len(s2), len([]rune(s2)))
}
```
>出力
```
7 7
15 5
```
## 文字へのアクセス
アルファベットは1文字1バイトですが、日本語はほぼ1文字3バイトです。
マルチバイト文字は、rune型に変換すると操作しやすくなります。
```go
package main

import (
	"fmt"
)

func main() {
	const (
		s1 string = "abcdefg"
		s2 string = "あいうえお"
	)

	fmt.Println("------------------------")
	fmt.Println(s1[0:1])
	fmt.Println(s1[:3])
	fmt.Println(s1[3:])
	fmt.Println(s1[3:4])

	fmt.Println("------------------------")
	fmt.Println(s2[:3])
	fmt.Println(s2[3:6])
	fmt.Println(s2[6:12])
	fmt.Println(s2[6:12])

	fmt.Println("------------------------")
	r := []rune(s2)
	fmt.Println(string(r[0:1]))
	fmt.Println(string(r[:3]))
	fmt.Println(string(r[3:]))
	fmt.Println(string(r[3:4]))
}
```
>出力
```
------------------------
a
abc
defg
d
------------------------
あ
い
うえ
うえ
------------------------
あ
あいう
えお
え
```
## エスケープシーケンス
```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("改行: aa\naa")
	fmt.Println("--------------------------")
	fmt.Println("水平タブ: aa\taa")
	fmt.Println("--------------------------")
	fmt.Println("垂直タブ: aa\vaa")
	fmt.Println("--------------------------")
	fmt.Println("ダブルクォーテーション: aa\"aa")
	fmt.Println("--------------------------")
	fmt.Println("バックスラッシュ: aa\\aa")
}
```
>出力
```
改行: aa
aa
--------------------------
水平タブ: aa    aa
--------------------------
垂直タブ: aa
            aa
--------------------------
ダブルクォーテーション: aa"aa
--------------------------
バックスラッシュ: aa\aa
```
## 文字列連結
+演算子 で連結できます。
```go
package main

import (
	"fmt"
)

func main() {
	s1 := "わくわく"
	s2 := "Bank"
	s := s1 + s2
	fmt.Println(s)
}
```
>出力
```
わくわくBank
```
# fmt package
## Print, Sprint
```go
package main

import "fmt"

func main() {
	fmt.Print("Hello World.\n")
	fmt.Print("aaa", "bbb", "\n")
	fmt.Print("aaa", "bbb", "ccc", "\n")

	s1 := fmt.Sprint("Hello World.\n")
	fmt.Print(s1)
	s2 := fmt.Sprint("aaa", "bbb", "\n")
	fmt.Print(s2)
	s3 := fmt.Sprint("aaa", "bbb", "ccc", "\n")
	fmt.Print(s3)
}
```
>出力
```
Hello World.
aaabbb
aaabbbccc
Hello World.
aaabbb
aaabbbccc
```
## Println, Sprintln
```go
package main

import "fmt"

func main() {
	fmt.Println("Hello World.")
	fmt.Println("aaa", "bbb")
	fmt.Println("aaa", "bbb", "ccc")

	s1 := fmt.Sprintln("Hello World.")
	fmt.Print(s1)
	s2 := fmt.Sprintln("aaa", "bbb")
	fmt.Print(s2)
	s3 := fmt.Sprintln("aaa", "bbb", "ccc")
	fmt.Print(s3)
}
```
>出力
```
Hello World.
aaa bbb
aaa bbb ccc
Hello World.
aaa bbb
aaa bbb ccc
```
## Printf, Sprintf
```go
package main

import "fmt"

func main() {
	s := "abcdef"
	fmt.Printf("value:%v type:%T\n", s, s)

	s1 := fmt.Sprintf("value:%v type:%T\n", s, s)
	fmt.Print(s1)
}
```
>出力
```
value:abcdef type:string
value:abcdef type:string
```
## フォーマット文字列(verb)
```go
package main

import "fmt"

type user struct {
	id   int
	name string
}

func main() {
	b := true
	fmt.Printf("boolean:%t\n", b)
	fmt.Println("-------------------------")

	i := 10
	fmt.Printf("2進数:%b\n", i)
	fmt.Printf("8進数:%o\n", i)
	fmt.Printf("10進数:%d\n", i)
	fmt.Printf("16進数:%x\n", i)
	fmt.Println("-------------------------")

	f := 3.1415
	fmt.Printf("%f\n", f)
	fmt.Printf("%.4f\n", f)
	fmt.Printf("%10.4f\n", f)
	fmt.Printf("%010.4f\n", f)
	fmt.Println("-------------------------")

	u := user{100, "wakuwaku"}
	fmt.Printf("%T\n", u)
	fmt.Printf("%v\n", u)
	fmt.Printf("%+v\n", u)
	fmt.Printf("%#v\n", u)
}
```
>出力
```
boolean:true
-------------------------
2進数:1010
8進数:12
10進数:10
16進数:a
-------------------------
3.141500
3.1415
    3.1415
00003.1415
-------------------------
main.user
{100 wakuwaku}
{id:100 name:wakuwaku}
main.user{id:100, name:"wakuwaku"}
```
# strings package
stringsパッケージで利用頻度の高そうなメソッドを抜粋します。Exampleが掲載されており、動作確認できます。

https://golang.org/pkg/strings/#Contains
https://golang.org/pkg/strings/#Count
特定文字が含まれている数をカウント
https://golang.org/pkg/strings/#Join
https://golang.org/pkg/strings/#Replace
https://golang.org/pkg/strings/#Split
https://golang.org/pkg/strings/#ToLower
https://golang.org/pkg/strings/#ToUpper
https://golang.org/pkg/strings/#Trim

# strconv package
strconvパッケージで利用頻度の高そうなメソッドを抜粋します。Exampleが掲載されており、動作確認できます。

https://golang.org/pkg/strconv/#Atoi
https://golang.org/pkg/strconv/#Itoa
#regexp package
Goの正規表現は遅いと言われており、まずはstringsパッケージで実装できないか検討することをお勧めします。

regexpパッケージで利用頻度の高そうなメソッドを抜粋します。Exampleが掲載されており、動作確認できます。

https://golang.org/pkg/regexp/#MustCompile
正規表現オブジェクトを作ります。正規表現を使いまわしたいとき活用。
https://golang.org/pkg/regexp/#Regexp.FindString
最初にマッチした文字列を取得
https://golang.org/pkg/regexp/#Regexp.FindAllString
マッチした文字列を全て取得
https://golang.org/pkg/regexp/#Regexp.MatchString
true, falseを返す
https://golang.org/pkg/regexp/#Regexp.ReplaceAllString
https://golang.org/pkg/regexp/#Regexp.Split

























