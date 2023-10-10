# Go言語による「テストの書き方・実行方法
>[Go言語でテストコードを書いてみよう](https://note.com/rescuenow_hr/n/n9ed7caf4646d)
>[Goテストシナリオケース集](https://future-architect.github.io/articles/20200601/#%E3%83%86%E3%82%B9%E3%83%88%E3%81%8C%E3%81%97%E3%81%9F%E3%81%84)

Table of Contents
=================
- [Go言語による「テストの書き方・実行方法](#go言語によるテストの書き方実行方法)
- [Table of Contents](#table-of-contents)
- [テストコードの書き方](#テストコードの書き方)
  - [テストコードの例](#テストコードの例)
- [テストファイルの場所](#テストファイルの場所)
- [テストの実行方法](#テストの実行方法)
- [テストのリポート（アサーション）](#テストのリポートアサーション)
- [テストをスキップしたい](#テストをスキップしたい)
      - [条件付けでテストをスキップする](#条件付けでテストをスキップする)
- [テストを並列に実施したい](#テストを並列に実施したい)
- [テストの前処理や後処理](#テストの前処理や後処理)
  - [TestMain例](#testmain例)
  - [後処理には `T.Cleanup` が便利](#後処理には-tcleanup-が便利)
    - [簡単な例](#簡単な例)
    - [T.Cleanup を使った例](#tcleanup-を使った例)
- [あるディレクトリ配下のテストをすべて実施したい](#あるディレクトリ配下のテストをすべて実施したい)
- [一部のテストケースのみ実施したい](#一部のテストケースのみ実施したい)
- [テストのキャッシュを削除したい](#テストのキャッシュを削除したい)
- [構造体、マップやスライスの比較を実施したい](#構造体マップやスライスの比較を実施したい)
- [テストデータを置いておきたい](#テストデータを置いておきたい)
- [テストにヘルパー関数を使いたい](#テストにヘルパー関数を使いたい)

# テストコードの書き方
Go言語のテストは testingパッケージ を利用します。
ファイル名と関数名の命名には下記の決まりがあります。

テストファイル名: xxx_test.go
テスト関数名: TestXxxもしくは Test_xxx（TestxxxはNG）
## テストコードの例
- TableDrivenTest とサブテストを組み合わせています。どちらも現場でよく使われます。
- サブテストを用いると各テストごとに結果がわかるようになります。
- TableDrivenTest はさまざまな Input/Output パターンを網羅するのに便利です。
>xxx_test.go
```go
import "testing"

func add(a, b int) int {
	return a + b
}

func TestXxx(*testing.T){
    type args struct {
		a int
		b int
	}
	tests := []struct {
		name string
		args args
		want int
	}{
		{
			name: "normal",
			args: args{a: 1, b: 2},
			want: 3,
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if got := add(tt.args.a, tt.args.b); got != tt.want {
				t.Errorf("add() = %v, want %v", got, tt.want)
			}
		})
	}
}
```
>出力
```bash
$ go test -v
=== RUN   TestAdd
--- PASS: TestAdd (0.00s)
PASS
```
# テストファイルの場所
テスト対象のファイルとテストファイルは同じディレクトリに配置するのを推奨されています。
- 別のフォルダに置くとカバレッジの取得ができなくなります。
```trre
sample_dir
 ├── xxx.go
 └── xxx_test.go
```
# テストの実行方法
テストの実行は `go test  ./dirctory…` で実行できます。 
- `…`をつけるとdirctory配下のディレクトリ下にあるテストファイルも実行します。
```bash
$ go test ./... 
```
- また、さまざまなオプションを指定することができます。
  - `-v`は詳細も表示する。
  - `--short`はフラグのあるテストファイルをskipする。(※後述)
  - `-cover`はカバレッジの取得ができます
# テストのリポート（アサーション）
Go はテストのアサーションを提供していません。
- 先程の TestXxx 関数のように、テストが失敗したことを開発者が自ら実装する必要があります。
- 失敗したことを示すには `T.Error (T.Errorf)` や `T.Fatal (T.Fatalf)` を用いることができます。
  - T.Fatal を用いると T.Fatal が実行された以降のテストは呼び出されずに終了します。
- テストが失敗したことを示すには T.Error を使い、
  - テストの初期化など、処理が失敗するとその後のテストが無意味になる場合は T.Fatal を用いる
- 以下のように t.Fatalf を用いた場合は、その後のテストの処理 t.Log("after add() ...") が呼び出されていないことが分かります
  - defer や T.Cleanup といった後処理は呼び出されます。
```go
func TestAdd(t *testing.T) {
	type args struct {
		a int
		b int
	}
	tests := []struct {
		name string
		args args
		want int
	}{
		{name: "fail", args: args{a: 1, b: 2}, want: 30},
		{name: "normal", args: args{a: 1, b: 2}, want: 3},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			// t.Fatalf でテストが失敗した場合でもクリーンアップ処理は呼び出される
			t.Cleanup(func() {
				t.Log("cleanup!")
			})

			// t.Fatalf でテストが失敗した場合でも defer の処理は呼び出される
			defer t.Log("defer!")

			if got := add(tt.args.a, tt.args.b); got != tt.want {
				t.Fatalf("add() = %v, want %v", got, tt.want)
			}
			// t.Fatalf でテストが失敗した場合は以下は呼び出されない
			t.Log("after add() ...")
		})
	}
}
```
>出力
```bash
$ go test -v
=== RUN   TestAdd
=== RUN   TestAdd/fail
    diff_test.go:31: add() = 3, want 30
    panic.go:617: defer!
    diff_test.go:24: cleanup!
=== RUN   TestAdd/normal
    diff_test.go:34: after add() ...
    diff_test.go:35: defer!
    diff_test.go:24: cleanup!
--- FAIL: TestAdd (0.00s)
    --- FAIL: TestAdd/fail (0.00s)
    --- PASS: TestAdd/normal (0.00s)
FAIL
exit status 1
FAIL    github.com/d-tsuji/go-sandbox 0.156s
```
!!! info なお t.Fatalf と似たような関数名でログ出力してアプリケーションを終了する log パッケージの Fatalf という関数があります。
    log.Fatalf のGo Docにもあるように log.Fatalf は defer といった後処理を呼び出さずに即座に os.Exit(1) でアプリケーションが終了します。
    t.Fatalf と log.Fatalf を混乱しないように注意しましょう。
# テストをスキップしたい
時間がかかるテストなど、自動テストなどではテストをスキップしたい場合があるかも
- その場合は func (c *T) Skip(args ...interface{}) というメソッドを用いることでスキップできます。
```go
for _, tt := range tests {
	t.Run(tt.name, func(t *testing.T) {
		// 以下を追加
		// ---------------------------------
		if testing.Short() {
			t.Skip("skipping test in short mode.")
		}
		// ---------------------------------
		if got := add(tt.args.a, tt.args.b); got != tt.want {
			t.Errorf("add() = %v, want %v", got, tt.want)
		}
	})
}
```
>出力
```bash
$ go test -v -short
=== RUN   Test_add
=== RUN   Test_add/normal_1
--- PASS: Test_add (0.00s)
    --- SKIP: Test_add/normal_1 (0.00s)
        main_test.go:35: skipping test in short mode.
PASS
ok      github.com/d-tsuji/go-sandbox   0.339s
```
テストがスキップされていることが分かります。
さらっと testing.Short() という関数も用いましたが Short() は testing パッケージに含まれている関数で、-short フラグがセットされていると true になります。
- そのためテストを実施するときに -short というフラグを付与したときだけテストがスキップされる、そうでないときはスキップされずテストが実施される、というように使い分けることができます。

#### 条件付けでテストをスキップする
標準パッケージでもテストのスキップが実装されているのを色々見ることができます。
以下は io/ioutil/ioutil_test.go からの抜粋です。
特定の条件を満たす場合にテストをスキップするように実装されています。

>io/ioutil/ioutil_test.go
```go
func TestReadOnlyWriteFile(t *testing.T) {
	if os.Getuid() == 0 {
		t.Skipf("Root can write to read-only files anyway, so skip the read-only test.")
	}
```
# テストを並列に実施したい
並列にテストを実施するには `func (t *T) Parallel()` メソッドを用いることができます。
上記のテストに `tt := tt` と `t.Parallel()` を追記します。
```go
for _, tt := range tests {
	// 以下を追加
	// ---------
	tt := tt
	// ---------
	t.Run(tt.name, func(t *testing.T) {
		// 以下を追加
		// -----------
		t.Parallel()
		// -----------
		if got := add(tt.args.a, tt.args.b); got != tt.want {
			t.Errorf("add() = %v, want %v", got, tt.want)
		}
	})
}
```
!!! info ループ時に割り当てているローカル変数 tt を捕捉することは重要です。Go ではループで用いられる変数は同じアドレスを使います。
```go
func main() {
	b := []byte("abcde")
	for i, c := range b {
		fmt.Printf("i: %#v, c: %#v ------- &i: %#v, &c: %#v\n", i, string(c), &i, &c)
	}
}
```
>出力
```bash
// 変数のアドレスがすべて同じアドレスを参照している
i: 0, c: "a" ------- &i: (*int)(0x40e020), &c: (*uint8)(0x40e024)
i: 1, c: "b" ------- &i: (*int)(0x40e020), &c: (*uint8)(0x40e024)
...
```
そのため、テストを並列に実行するときは tt := tt などとして変数をシャドウイングし、並列で実行しているテストに影響がないようにする必要があります。

ただし、テストケース自体がそもそも並列に実行できない場合、例えばデータベース上のテーブルへの UPDATE や INSERT が発生し、テストケースで競合する場合、テストを並列に実行することはできないため、注意が必要です
# テストの前処理や後処理
テストをしていると、前処理や後処理をしたい場合があると思います。
主な例の 1 つとしてデータベースの処理化があるでしょう。
- テストを実施する前処理としてあるデータを INSERT しておいて、テスト実施後に対象のテーブルのデータを削除する、といったものです。

そのような共通的な前処理や後処理を実施したい場合は`func TestMain(m *testing.M)`関数を用いることができます。
## TestMain例
以下のように TestMain を用いてテストを制御するとテストの前後(今回の場合は テスト対象関数() の前後)に処理を実行できます。m.Run() の実行結果を取得して os.Exit() するのが慣用的です。
```go
func Test_f(t *testing.T) {
	テスト対象関数()
}

func TestMain(m *testing.M) {
	fmt.Println("前処理")
	status := m.Run()
    fmt.Println("後処理")

    os.Exit(status)
}
```
>出力
```bash
$ go test
前処理
なんらかの処理
PASS
後処理
ok      github.com/d-tsuji/go-sandbox   0.298s
```
## 後処理には `T.Cleanup` が便利
Go1.14 でテスト時に生成したリソースを便利に後処理できる関数が登場しました。T.Cleanup です。
- `func (c *T) Cleanup(f func())`

T.Cleanup を便利に使えることが実感できるシーンの 1 つとして、テストに必要な前処理をテストとは別の関数で実施している場合があります。
### 簡単な例
テストの前準備としてテスト用のファイルを生成する必要があったとして、テスト終了後に削除したい場合、以下のような実装が考えられます。
TempFile 関数ではリソースをクローズする処理を呼び出し元に返却する必要があり、呼び出し元で後処理として teardown 関数を呼び出すことになります。
>testutil/file.go
```go
func TempFile(t *testing.T, content []byte) (name string, teardown func()) {
	file, err := ioutil.TempFile("", "test")
	if err != nil {
		t.Error(err)
	}

	if err = ioutil.WriteFile(file.Name(), content, 0644); err != nil {
		t.Error(err)
	}

	return file.Name(), func() {
		syscall.Unlink(file.Name())
	}
}
```
>呼び出し元の処理
```go
file, delete := testutil.TempFile(t, nil /* something */)
defer delete()
```
### T.Cleanup を使った例
T.Cleanup を用いると前処理を実施する関数内でリソースの後処理が実施できるようになります。
TempFile 関数の例であれば、以下のように t.Cleanup を用いることができます。
- 関数から return したタイミングで呼び出される defer とは異なり、テストが完了したタイミングで Cleanup 処理が呼び出されます。
```go
func TempFile(t *testing.T, content []byte) string {
	file, err := ioutil.TempFile("", "test")
	if err != nil {
		t.Error(err)
	}
	t.Cleanup(func() { syscall.Unlink(file.Name()) })

	if err = ioutil.WriteFile(file.Name(), content, 0644); err != nil {
		t.Error(err)
	}

	return file.Name()
}
```
# あるディレクトリ配下のテストをすべて実施したい
たとえば標準パッケージの例だと、io パッケージはテスト対象に含めるが、その他のパッケージはテスト対象に含めない…といった要領です。
- これはテストのコマンドというよりはパッケージのコマンドになります。

以下のように `...` の文字列を用いてワイルドカードとしてテスト対象のファイルを選択できます。詳細は go help packages とすることで確認できます。

>io パッケージに含まれるすべてのテストを実行します。
```bash
go test io/...
```
同様に io パッケージに含まれる ioutil パッケージのみテストしたい場合は以下のようになります。
```bash
$ go test -v io/ioutil/...
=== RUN   TestReadFile
--- PASS: TestReadFile (0.00s)
=== RUN   TestWriteFile
--- PASS: TestWriteFile (0.00s)
...
```
# 一部のテストケースのみ実施したい
実行対象のテストを抽出するには -run フラグを用いることできます。
>以下のように正規表現を用いて、一致するテストのみを実行できます。
```bash
-run 正規表現
```
>以下のように実行するとテストの関数名に File が含まれるテストのみ実行されていることが分かります。
```bash
$ go test -v -run File
=== RUN   TestReadFile
--- PASS: TestReadFile (0.00s)
PASS
...
```
# テストのキャッシュを削除したい
キャッシュ使わない場合はを明示的に -count=1 と指定すればよいです。
- `-count=1` と明示的に指定するとテストはキャッシュされなくなります。
```bash
$ go test -v -count=1
=== RUN   TestReadFile
--- PASS: TestReadFile (0.00s)
PASS
...
```
`go clean -cache` を用いてもビルドキャッシュ全体を削除することができ、テストのキャッシュも削除できます。
# 構造体、マップやスライスの比較を実施したい
map のキーと値が一致しているかどうか確認するようなテストがしたいとしましょう。map や slice は spec#Comparison_operators にもあるように比較演算子を用いて比較することができません。

reflect.DeepEqual を使った同値チェックは google/go-cmp を使うとより便利にテストができるので、私は reflect.DeepEqual の代わりとして google/go-cmp を用いることが多いです。
```go
func TestMakeGatewayInfoGoCmp(t *testing.T) {
	got, want := MakeGatewayInfo()
	if diff := cmp.Diff(want, got); diff != "" {
		t.Errorf("MakeGatewayInfo() mismatch (-want +got):\n%s", diff)
	}
}
```
go-cmp の結果は何が同値で、何が同値でなかったか、同値でなかったときは取得した値と想定する値は何か明示的にわかるのがよいです。他にもオプションで条件をカスタマイズできます。(やりすぎ注意)

社内でも stretchr/testify/assert を使う勢なども見かけます。Go のテスティングフレームに関する話は好みが分かれるところだと思うので、深くは触れません。
# テストデータを置いておきたい

テストの Input や Output になるファイルを testdata という名前のディレクトリに置いておくことができます。testdata ディレクトリに含まれるファイルはテストのときのみ用いられます。
- 標準パッケージの中でたくさん用いられていますが、image パッケージの例を上げると https://github.com/golang/go/tree/master/src/image/testdata といったものです。
# テストにヘルパー関数を使いたい
複数のテストで用いるような共通の関数をヘルパー関数として実装する場合があると思います。テスト用のヘルパー関数の特徴は以下です。
- ヘルパー関数はエラーを返さない
- *testing.T を受け取ってテストを落とす
- Go 1.9 からは T.Helper を使って情報を補足する
  - https://tip.golang.org/pkg/testing/#T.Helper
  - t.Helper() を付けた場合のほうが、テストケース内のどの行で失敗したか分かりやすくなり、エラーの原因を探りやすくなる

[Go Friday 傑作選](https://www.slideshare.net/takuyaueda967/go-friday) より引用

テスト用のヘルパー関数は以下のように実装されます。
```go
func mustUrlParse(t *testing.T, s string) *url.URL {
	t.Helper()
	u, err := url.Parse(s)
	if err != nil {
		t.Fatal(err)
	}
	return u
}
```







































