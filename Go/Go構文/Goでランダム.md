# Golang で math/rand を使用して固定長のランダム文字列を生成する
>[参考記事](https://www.delftstack.com/ja/howto/go/generate-random-string-of-fixed-length-in-golang/)

```go
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    fmt.Println(RandomString(30))
}

func RandomString(n int) string {
    var letterRunes = []rune("abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890")

    b := make([]rune, n)
    for i := range b {
        b[i] = letterRunes[rand.Intn(len(letterRunes))]
    }
    return string(b)
}
```
>出力：
```bash
XVlBzgbaiCMRAjWwhTHctcuAxhxKQF
```