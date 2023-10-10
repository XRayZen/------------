# Go言語の概要
Go言語はGoogle社が開発したオープンソースのプログラミング言語で、C言語に似た文法を持ちながらもメモリ管理や並行処理を簡単に扱うことができるように設計されています。また、コンパイル型言語であるため、高速に動作することが特徴です。

以下にGo言語の主な特徴をいくつか挙げてみます。

- シンプルな文法  
  Go言語は、C言語やJavaといった言語に似た文法を持っていますが、冗長な機能が省かれており、シンプルな文法のため読みやすく書きやすいという利点があります。

- メモリ管理  
  Go言語は、自動的にメモリを解放するガベージコレクション機能を持っています。
  - これにより、開発者が手動でメモリ管理をしなくても良いため、安全性や生産性が向上します。

- 並行処理  
  Go言語は、チャネルと呼ばれる仕組みを使って、簡単に並行処理を扱うことができます。これにより、複数の処理を同時に実行する場合にも問題なく動作するため、高い拡張性を持っています。

- 高速な実行  
  Go言語は、コンパイル型言語であるため、C言語やJavaと同様に高速な実行が可能です。また、並行処理に対しても最適化されているため、多重化されたタスクを適切に管理しながら高いパフォーマンスを発揮します。

以上がGo言語の主な特徴です。これらの特徴により、Webサーバやネットワークアプリケーション、分散システムの開発などに適した言語とされています。

Table of Contents
=================
- [Go言語の概要](#go言語の概要)
- [Table of Contents](#table-of-contents)
- [Goのインストール・環境変数・コマンド](#goのインストール環境変数コマンド)
  - [インストール](#インストール)
  - [環境変数( go env )](#環境変数-go-env-)
  - [GOROOT](#goroot)
  - [GOPATH](#gopath)
- [Hello Worldの確認](#hello-worldの確認)
  - [go build](#go-build)
- [go modによるモジュール管理](#go-modによるモジュール管理)
  - [go mod init](#go-mod-init)
  - [go mod tidy](#go-mod-tidy)
  - [モジュールの削除](#モジュールの削除)
  - [go getとgo install](#go-getとgo-install)
    - [go install の使い方](#go-install-の使い方)

# Goのインストール・環境変数・コマンド
>[記事](https://www.wakuwakubank.com/posts/775-go-install-env-command/)

Macにgoをインストールして、簡単な動作確認をします。
GOROOT・GOPATHの役割、goコマンド(env, get, build, run, mod)、GoLandの使い方について確認

## インストール
```bash
brew install go
```
## 環境変数( go env )

## GOROOT
Goがインストールされた場所は、GOROOT で確認できます
```bash
go env | grep GOROOT
GOROOT="/usr/local/Cellar/go/1.15.6/libexec"
```
## GOPATH
外部から取得したパッケージの格納先を決める際、GOPATHの影響を受けます。
```bash
go env | grep GOPATH

>> GOPATH="/Users/xxx/go"
```
# Hello Worldの確認
main.goファイルを作り、以下処理を記述します。
```go
package main

import "fmt"

func main() {
  fmt.Println("Hello world")
}
```
コンパイルして実行します。
- go run を利用するとビルドと実行を同時に行えます。
```bash
go run main.go
```
## go build
go build で実行バイナリを生成できます。
```bash
$ go build main.go   
$ ls -l
total 4200
-rwxr-xr-x  1 xxx  staff  2142872  Dec  4 02:32 main
-rw-r--r--  1 xxx  staff       73  Dec  4 02:32 main.go
```
生成された実行バイナリを起動してみます。
```bash
$ ./main 
Hello world
```
# go modによるモジュール管理
>https://zenn.dev/yoonchulkoh/articles/9729d9e1304738

Go言語の 1.11以上 で go mod が使えるようになりました。
依存モジュールを go.modファイル と go.sumファイル で管理できます。

go mod では以下のサブコマンドを利用できます。
```bash
$ go help mod
```
## go mod init
go mod init で初期化します。
```bash
$ go mod init example.com/xxx
```
go.modファイルが作成されました。
```bash
$ cat go.mod 
module example.com/xxx

go 1.15
```
## go mod tidy
main.goに以下処理を記述します。
```go
package main

import (
        "net/http"

        "github.com/labstack/echo/v4"
)

func main() {
        // Echo instance
        e := echo.New()

        // Routes
        e.GET("/", hello)

        // Start server
        e.Logger.Fatal(e.Start(":1323"))
}

// Handler
func hello(c echo.Context) error {
        return c.String(http.StatusOK, "Hello, World!")
}
```
echoパッケージを利用しています。

go mod tidy を利用すると、依存モジュールを自動インストールして、使われていない依存モジュールを削除してくれます。
```bash
$ go mod tidy
```
## モジュールの削除
前述したgo mod tidyを使う。
最初、go.modからモジュールを削除してgo mod tidy叩いてみたけど消えず、しばらく悩みました。

正しいやり方は、
1. コードから不要なモジュールのimport文を削除する
2. go mod tidyを叩く 

## go getとgo install
goの1.16から、go get と go install の使い分けがわかりやすくなりました。
- go get は、go.modの編集に利用します。
- go install は、ツールのグローバルインストールに利用します。
### go install の使い方
以下の形式で利用できます。
```bash
go install example.com/cmd@v1.0.0
```
- バージョン指定は必須です。最新のバージョンを指定したい場合 @latest をつけます。

GOPATH が設定されている場合、$GOPATH/bin 配下に実行バイナリがインストールされます。
```bash
$ go install github.com/tsenart/vegeta@latest
go: downloading github.com/tsenart/vegeta v1.2.0
  (略)
$ 
$ 
$ ls $GOPATH/bin
vegeta
```





