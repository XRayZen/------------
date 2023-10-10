# JSONデータのエンコード・デコード(Marshal, Unmarshal)
>[JSONデータのエンコード・デコード(Marshal, Unmarshal)](https://www.wakuwakubank.com/posts/794-go-json/)

jsonパッケージを利用して、JSONデータの「エンコード」と「デコード」の動作確認を行います。
エンコードにはMarshal関数を利用して、デコードにはUnmarshal関数を利用します。

Table of Contents
=================
- [JSONデータのエンコード・デコード(Marshal, Unmarshal)](#jsonデータのエンコードデコードmarshal-unmarshal)
- [Table of Contents](#table-of-contents)
- [Marshal (エンコード)(Go → JSON)](#marshal-エンコードgo--json)
  - [sliceをエンコード](#sliceをエンコード)
  - [mapをエンコード](#mapをエンコード)
  - [structをエンコード(フィールド定義にタグ指定)](#structをエンコードフィールド定義にタグ指定)
- [Unmarshal (デコード)(JSON → Go)](#unmarshal-デコードjson--go)
  - [Jsonをデコード](#jsonをデコード)
    - [読み込むJSONファイル](#読み込むjsonファイル)
    - [デコードするGoコード](#デコードするgoコード)

# Marshal (エンコード)(Go → JSON)
json.Marshal でGoの値をJSONデータにエンコードして、ファイル出力してみます。

## sliceをエンコード
```go
func main() {
	s := []string{"aaa", "bbb", "ccc"}

	sj, _ := json.Marshal(s)
	fmt.Printf("sj=%+v\n\n", sj)
	fmt.Printf("string(sj)=%+v\n\n", string(sj))

	if err := ioutil.WriteFile("slice.json", sj, 0666); err != nil {
		log.Fatal(err)
	}
}
```
>実行結果
```
$ go run main.go 
sj=[91 34 97 97 97 34 44 34 98 98 98 34 44 34 99 99 99 34 93]

string(sj)=["aaa","bbb","ccc"]
```
## mapをエンコード
```go
import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
)

func main() {
	m := map[string]interface{}{
		"aaa": 10,
		"bbb": 20,
		"ccc": []int{100, 200, 300},
	}

	mj, _ := json.Marshal(m)
	fmt.Printf("mj=%+v\n\n", mj)
	fmt.Printf("string(mj)=%+v\n\n", string(mj))

	if err := ioutil.WriteFile("map.json", mj, 0666); err != nil {
		log.Fatal(err)
	}
}
```
>実行結果
```
$ go run main.go 
mj=[123 34 97 97 97 34 58 49 48 44 34 98 98 98 34 58 50 48 44 34 99 99 99 34 58 91 49 48 48 44 50 48 48 44 51 48 48 93 125]

string(mj)={"aaa":10,"bbb":20,"ccc":[100,200,300]}
```
## structをエンコード(フィールド定義にタグ指定)
構造体(struct)をエンコードします。
JSONデータにエンコードされるフィールドですが、頭文字が大文字のフィールドが対象になります。
また、タグ指定で以下調整が可能です。
- フィールド名を変更
- omitempty をつけることで nil のときフィールドを省略
```go
type User struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

type Post struct {
	Title         string   `json:"post_title"`
	Tags          []string `json:"tags"`
	User          User     `json:"user"`
	Memo          *string  `json:"memo"`
	MemoOmitEmpty *string  `json:"memo_omit_empty,omitempty"`
	privateMemo   *string
}

func main() {
	p := Post{
		Title: "HelloWorld",
		Tags:  []string{"aaa", "bbb"},
		User: User{
			Name: "wakuwaku",
			Age:  41,
		},
	}
	fmt.Printf("p=%+v\n\n", p)

	pj, _ := json.Marshal(p)
	fmt.Printf("pj=%+v\n\n", pj)
	fmt.Printf("string(pj)=%+v\n\n", string(pj))

	if err := ioutil.WriteFile("struct.json", pj, 0666); err != nil {
		log.Fatal(err)
	}
}
```
>実行結果
```
$ go run main.go 
p={Title:HelloWorld Tags:[aaa bbb] User:{Name:wakuwaku Age:41} Memo:<nil> MemoOmitEmpty:<nil> privateMemo:<nil>}

pj=[123 34 112 111 115 116 95 116 105 116 108 101 34 58 34 72 101 108 108 111 87 111 114 108 100 34 44 34 116 97 103 115 34 58 91 34 97 97 97 34 44 34 98 98 98 34 93 44 34 117 115 101 114 34 58 123 34 110 97 109 101 34 58 34 119 97 107 117 119 97 107 117 34 44 34 97 103 101 34 58 52 49 125 44 34 109 101 109 111 34 58 110 117 108 108 125]

string(pj)={"post_title":"HelloWorld","tags":["aaa","bbb"],"user":{"name":"wakuwaku","age":41},"memo":null}
```
# Unmarshal (デコード)(JSON → Go)
JSONファイルを読み込み、json.Unmarshal でJSONデータをGoの値にデコードしてみます。

## Jsonをデコード
### 読み込むJSONファイル
ここでは、下記JSONファイルを利用します。
```json
[
  {
    "post_title": "Hello",
    "tags": [
      "aaa",
      "bbb"
    ],
    "user": {
      "name": "wakuwaku",
      "age": 41
    }
  },
  {
    "post_title": "World",
    "tags": [],
    "user": {
      "name": "わくわく",
      "age": 27
    },
    "memo": "abcdefg"
  }
]
```
### デコードするGoコード
```go
import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
)

type User struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

type Post struct {
	Title       string   `json:"post_title"`
	Tags        []string `json:"tags"`
	User        User     `json:"user"`
	Memo        *string  `json:"memo"`
	privateMemo *string
}

func main() {
	bytes, err := ioutil.ReadFile("struct.json")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("bytes=%+v\n\n", bytes)
	fmt.Printf("string(bytes)=%+v\n\n", string(bytes))

	var posts []Post
	if err := json.Unmarshal(bytes, &posts); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("posts=%+v\n\n", posts)
}
```
>実行結果
```
$ go run main.go 
bytes=[91 10 32 32 123 10 32 32 32 32 34 112 111 115 116 95 116 105 116 108 101 34 58 32 34 72 101 108 108 111 34 44 10 32 32 32 32 34 116 97 103 115 34 58 32 91 10 32 32 32 32 32 32 34 97 97 97 34 44 10 32 32 32 32 32 32 34 98 98 98 34 10 32 32 32 32 93 44 10 32 32 32 32 34 117 115 101 114 34 58 32 123 10 32 32 32 32 32 32 34 110 97 109 101 34 58 32 34 119 97 107 117 119 97 107 117 34 44 10 32 32 32 32 32 32 34 97 103 101 34 58 32 52 49 10 32 32 32 32 125 10 32 32 125 44 10 32 32 123 10 32 32 32 32 34 112 111 115 116 95 116 105 116 108 101 34 58 32 34 87 111 114 108 100 34 44 10 32 32 32 32 34 116 97 103 115 34 58 32 91 93 44 10 32 32 32 32 34 117 115 101 114 34 58 32 123 10 32 32 32 32 32 32 34 110 97 109 101 34 58 32 34 227 130 143 227 129 143 227 130 143 227 129 143 34 44 10 32 32 32 32 32 32 34 97 103 101 34 58 32 50 55 10 32 32 32 32 125 44 10 32 32 32 32 34 109 101 109 111 34 58 32 34 97 98 99 100 101 102 103 34 10 32 32 125 10 93]

string(bytes)=[
  {
    "post_title": "Hello",
    "tags": [
      "aaa",
      "bbb"
    ],
    "user": {
      "name": "wakuwaku",
      "age": 41
    }
  },
  {
    "post_title": "World",
    "tags": [],
    "user": {
      "name": "わくわく",
      "age": 27
    },
    "memo": "abcdefg"
  }
]

posts=[{Title:Hello Tags:[aaa bbb] User:{Name:wakuwaku Age:41} Memo:<nil> privateMemo:<nil>} {Title:World Tags:[] User:{Name:わくわく Age:27} Memo:0xc000010400 privateMemo:<nil>}]
```



































