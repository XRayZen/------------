# Go並列処理
>[【Go言語入門】goroutineとは？ 実際に手を動かしながら goroutineの基礎を理解しよう！](https://www.ariseanalytics.com/activities/report/20221005/)

Table of Contents
=================
- [Go並列処理](#go並列処理)
- [Table of Contents](#table-of-contents)
- [goroutineとは](#goroutineとは)
- [goroutine実装](#goroutine実装)
- [channel](#channel)
  - [channelとは](#channelとは)
  - [channelの宣言](#channelの宣言)
    - [実行順序](#実行順序)
  - [channelの方向](#channelの方向)
    - [buffered channel](#buffered-channel)
    - [close](#close)
    - [for range](#for-range)
    - [select](#select)
- [同期化オブジェクト](#同期化オブジェクト)
  - [Mutex](#mutex)
      - [Mutexを使用せず、2個のgoroutineがそれぞれ1万回ずつスライスに値を追加するコード](#mutexを使用せず2個のgoroutineがそれぞれ1万回ずつスライスに値を追加するコード)
        - [結果](#結果)
      - [上のコードでスライスをMutexで保護したコード](#上のコードでスライスをmutexで保護したコード)
        - [結果](#結果-1)
  - [WaitGroup](#waitgroup)
    - [syncパッケージで提供するWaitGroupの構造体と関数](#syncパッケージで提供するwaitgroupの構造体と関数)
    - [WaitGroupを使用して、すべてのgoroutineが終った後mainを終了するコード](#waitgroupを使用してすべてのgoroutineが終った後mainを終了するコード)
        - [結果・解説](#結果解説)
- [goroutineを使用する時の注意点](#goroutineを使用する時の注意点)
  - [Never start a goroutine without knowing how it will stop](#never-start-a-goroutine-without-knowing-how-it-will-stop)
- [まとめ](#まとめ)


# goroutineとは
goroutineは「Goでプログラムの同時性を簡単に具現し、既存の単純スレッド基盤に比べて効率的な動作を遂行するために作った作業単位」です。
全てのGoプログラムは必ず1個以上のgoroutineを持ち、常にバックグラウンドで動作します。
それぞれのgoroutineは独立的に実行されます。
goroutineの特徴は非常に軽量なスレッドということです。
これについては下に大きく3つに分けて説明します。
- メモリ消費
  - goroutineはカーネルスレッド（以下、スレッド）に比べてより少ないメモリのみ必要です。
  - スレッドはスレッド間のメモリ保護の役割をするGuard pageスペースを含めて1MB程度のスタックを必要とします。
  - 一方、goroutineは2KBのスタックだけが必要です。
    - そして、メモリが足りない場合はヒープを使用します。
- 生成・破壊コスト
  - スレッドはOSからリソースを要請し、作業が終了したらリソースを戻すなどやり取りの時間がかかります。
  - しかし、goroutineはGo Runtimeから生成と破壊が行われるためコストが低いです。
- コンテキストスイッチ
  - goroutineはスレッドモデルとしてM:Nモデルを採用しています。
    - そのため、マルチタスキングによるコンテキストスイッチが抑えられ、マルチコアが活用できるというメリットがあります。
  - スケジューリングが複雑になるというデメリットがありますが、GO Runtime Schedulerが解消してくれます。
    - ちなみに、Goはgoroutineの受動管理をサポートしていないです。
- Go Runtime Scheduler？
  - Go Runtime Schedulerは、Goプログラムが実行される時点で一緒に実行されてgoroutineを効率的にスレッドにスケジューリングさせる役割を遂行します。
  - 下のような原則を持って動作します。
    - カーネルスレッドはコストが高いため、できる限り少なく使用する。
    - 多くのgoroutineを実行し、高いコンカレンシーを維持する。
    - Nコアマシンで、N個のgoroutineをパラレルに動作させる。

!!! info 一番下の原則からgoroutineはマルチCPUコアを持ったハードウェアで最も性能を出せるということが分かります。

# goroutine実装
goroutineの作り方はすごく簡単です。
関数の前にgoというキーワードを付けて、その関数を実行するだけでgoroutineが生成されます。
```go
go function()
```
無名関数を利用することも可能です。
```go
 go func() {
    ...  
  }()
```
下はgoroutineを実装したコードです。どのような結果になるか考えてみてください。
```go
package main

import (
  "fmt" //標準I/Oのためにfmtパッケージをimport
)

func Say(s string) {
  fmt.Println(s)
}

func main() {

  go Say("hello")
  go Say("world")

}
```
上記の「全てのGoプログラムは必ず1個以上のgoroutineを持つ」という内容がヒントになります。これは、main関数も1つのgoroutineだということです。つまり、このmain goroutineの上で動くgoroutineは自分を呼び出した関数が無くなったことで消滅され、何も出力されない結果になります。

理由が分かったので、結果が見えるようにしてみましょう。単純にmain関数の終了を遅延させれば結果が見えるはずです。
```go
... // 同じコード

func main() {

	go Say("hello")
	go Say("world")

	fmt.Scanln()
}
```
方法はいろいろありますが、ここではScanln()を呼び出し、ユーザーの入力を待つように実装することでmain関数を終了しないようにしました。

実行すると、
```bash
-- A --
hello
world

-- B --
world
hello
```
出力結果はAかBのどちらかになります。実行を繰り返してみると結果が変わることが分かります。goroutineはgoroutine同士が独立しているため、実行の順序性は担保されていません。

では、どうやって実行の順序性（実行フロー）をgoroutineは制御できるのでしょうか。
- この時使うのが`channel`です。
# channel 
## channelとは
channelはgoroutine同士の値をやり取りする通路の役割と実行フローを制御する役割を担うデータ構造です。 channel自体はvalueではなく、Referenceタイプです。そして、全てのタイプをchannelとして使用ができ、channelに値を渡して抽出する形で使用します。Goはchannelを基本データ型で提供するので、他のパッケージやライブラリなしですぐ使用することができます。

channelを図で表現すると下のようになります。

![Alt text](https://www.ariseanalytics.com/wp-content/uploads/2022/10/05144212/324e77e1-9b0a-4981-82e1-7517fb7f64f2-1.png)
## channelの宣言
channelはmake()で生成できます。
```go
ch := make(chan string)
```
値は<-でやり取りができます。
- ch <- data    // channelにdata変数の値を送る（送信）
- data := <-ch  // channelから値を抽出し、その値をdata変数に入れる（受信）

では、channelを使用してみましょう。
```go
package main

import (
	"fmt"
)

func Say(c chan string) {
	data := <-c // A
	fmt.Println(data)

	data = "Let's Go"
	c <- data // B
}

func main() {

	ch := make(chan string)

	go func(c chan string) {
		data := "hello world"
		c <- data // C

		data = <-c // D
		fmt.Println(data)
	}(ch)

	go Say(ch)

	fmt.Scanln()
}
```
その結果は
```bash
hello world
Let's Go
```
先に「hello world」が出力された後「Let’s Go」が出力されます。この結果は何回を実行しても変わりません。何故なのかは実行順序を見ながら説明したいと思います。
### 実行順序
1. C （※AとCの実行順位が同じ時点）
・最初、2個のgorountineの中でどちらが先に実行されるかは分かりません。
・しかし、どちらが先に実行されるとしても「A」は受信なので、channelに値が送信されることを待ちます。
・そのため、いずれの時も「C」が先に実行されます。
2. A
・「C」がchannelに値を送信した後は次の行に進まず、channelの値が受信されることを待ちます。
・待っていた「A」はchannelの値を受信します。
3. B（※BとDの実行順位が同じ時点）
・「A」がchannelの値を受信した後、そのまま進んで「B」まで実行されます。
・「A」が受信する時点で「C」の待ち状態が解除されるため「D」も実行されます。
・つまり、この時点では「B」と「D」の中でどちらが先に実行されるかは分かりません。
・しかし、どちらが先に実行されるとしても「D」は受信なので、channelに値が送信されることを待ちます。
・そのため、いずれの時も「B」が先に実行されます。
4. D
・「B」がchannelに値を送信した後は次の行に進まず、channelの値が受信されることを待ちます。
・待っていた「D」がchannelの値を受信します。

上の流れからchannelを利用することでgoroutine同士に値のやり取りができること、goroutineの実行順序を制御できることが分かります。これでchannelについて理解できたと思います。次は、Goでchannelを操作する方法を軽く見てみましょう。
## channelの方向
基本的にchannelは双方向ですが、単方向channelも作ることができます。
単方向チャンネルを作る時はmake()と矢印を使用します。
```go
// 受信用channel
c1 := make(<-chan Type)

// 送信用channel
c2 := make(chan<- Type)
```
### buffered channel
buffered channelは、受信者が受け取る準備ができていなくても、指定されたバッファだけ値を送信し、継続して他のタスクを遂行することができます。buffered channelで送信側はバッファがいっぱいになった場合にのみ遮断され、受信側ではバッファが空いている場合にのみ遮断されます。

bufferd channelは、make(chanType、N)を使用して作成することができます。Nには使用するバッファの個数を書きます。
```go
myChannel := make(chan Type, N)
```
### close
close()を使用してchannelを閉めることができます。channelを閉めたら、該当channelには二度と送信することはできません。しかし、channelに値が存在する限り受信は可能です。
```go
close(myChannel)
```
下のコードを使用してchannelが閉じているかどうか確認することができます。閉じていたらcheckがfalseになり、開いていたらcheckがtrueになります。
```go
data, check := <-myChannel
```
### for range
for rangeを使用してchannelが閉じる時まで値を受信することができます。channelが開いていたらrangeはchannelに値が入るまで待機します。channelが閉じられたらループは終了になります。
```go
for data := range myChannel {
  ...
}
```
### select
switchと似ていますが、selectでcaseはchannelで送信または受信作業を意味します。selectはcaseのいずれかが実行されるまで待機します。 もし多数のcaseが用意される場合には、selectがランダムで一つを選択します。selectにdefaultがあれば、caseが用意されていなくても待機せずにdefaultを実行します。
```go
select {
  case <-ch1:
    // ch1に値が入った時に実行
  case <-ch2:
    // ch2に値が入った時に実行
  default:
    // 全てのchannelに値が入らなかった時に実行
}
```
# 同期化オブジェクト
Goではchannel以外にもgoroutineの実行フローを制御する同期化オブジェクトを提供します。
オブジェクトは複数ありますが、本記事ではMutexとWaitGroupについて話します。

## Mutex
Mutexは複数のgoroutineが共有する値を保護する時に使用します。
syncパッケージで提供するMutexの構造体と関数は次のようです。
- sync.Mutex
- func (m *Mutex) Lock()
- func (m *Mutex) Unlock()

#### Mutexを使用せず、2個のgoroutineがそれぞれ1万回ずつスライスに値を追加するコード
```go
package main

import (
	"fmt"
	"time" // Sleepを利用してプログラムを待機させるためにtimeパッケージをimport
)

func main() {

	var data = []int{}

	go func() {
		for i := 0; i < 10000; i++ {
			data = append(data, 1)
		}
	}()

	go func() {
		for i := 0; i < 10000; i++ {
			data = append(data, 1)
		}
	}()

	time.Sleep(2 * time.Second) // 2秒待機

	fmt.Println(len(data)) // スライスの長さを出力
}
```
##### 結果
結果として20000を期待しますが、複数回実行してみると20000、10000、9432、13425…など結果が20000だと担保されていないことが分かります。
- これは二つのgoroutineが競合し、同時に値にアクセスしたのでappend()が正確に処理されていない時があるためです。

#### 上のコードでスライスをMutexで保護したコード
```go
package main

import (
	"fmt"
	"sync" // Mutexオブジェクトを使用するためにsyncパッケージをimport
	"time" // Sleepを利用してプログラムを待機させるためにtimeパッケージをimport
)

func main() {

	var data = []int{}
	var mutex = new(sync.Mutex)

	go func() {
		for i := 0; i < 10000; i++ {
			mutex.Lock() // スライスを保護
			data = append(data, 1)
			mutex.Unlock() // スライスを保護解除
		}
	}()

	go func() {
		for i := 0; i < 10000; i++ {
			mutex.Lock() // スライスを保護
			data = append(data, 1)
			mutex.Unlock() // スライスを保護解除
		}
	}()

	time.Sleep(2 * time.Second) // 2秒待機

	fmt.Println(len(data)) // スライスの長さを出力
}
```
##### 結果
複数回実行してみるといつも20000という結果が期待通りに出力されます。これでMutexによってスライスが保護され、append()が正確に処理されたことが分かります。

ただし、Lock()とUnlock()は必ずペアを合わせなければならず、ペアが合わない場合はデッドロック(deadlock)が発生するので注意しましょう。
## WaitGroup
WaitGroupは、goroutineがすべて終わるまで待つ時に使用します。 前ではtime.Sleep()やfmt.Scanln()を使用してgoroutineが終わるまで臨時待機しました。 今回はWaitGroupを使用し、goroutineが終わるまで待ってみます。

### syncパッケージで提供するWaitGroupの構造体と関数
- sync.WaitGroup
- func (wg *WaitGroup) Add(delta int)
- func (wg *WaitGroup) Done()
- func (wg *WaitGroup) Wait()

### WaitGroupを使用して、すべてのgoroutineが終った後mainを終了するコード
```go
package main

import (
	"fmt"
	"sync"
)

func main() {

	wg := new(sync.WaitGroup)

	for i := 0; i < 5; i++ {
		wg.Add(1) // 繰り返す度にAdd関数で1ずつ追加
		go func(n int) {
			fmt.Println(n)
			wg.Done() // goroutineが終わったこと知らせる
		}(i)
	}

	wg.Wait() // 全てのgoroutineが終わるまで待機
	fmt.Println("プログラム終了")
}
```
##### 結果・解説
Add()でgoroutineの数を追加し、goroutineの中でDone()を使用してgoroutineが終了したことを知らせます。そして、Wait()ですべてのgoroutineが終わるまで待ちます。なので、実行してみると5個のgoroutineが終った後mainが終了されることが分かります。

ただし、Add()に設定した値とDone()が呼び出される回数は同じである必要があります。 この回数が合わないとpanicになるので注意しましょう。
# goroutineを使用する時の注意点
下はGoコミッターであるDave氏が話したことです。
## Never start a goroutine without knowing how it will stop
- 翻訳すると、「どのように停止するかを知らずにゴルーチンを開始しないでください」ということです。
- 本記事で使用したコードは単純なのでどのタイミングでどのようにgoroutineが終るかを分かりやすいでが、業務で使用することになったら複雑でgoroutineのライフサイクルが分かりにくくなると思います。

この時はGoのcontext（WithCancel、WithDeadlineなど）を使用することでgoroutineのライフサイクルを制御することができます。興味がある方はぜひ調べてみてください。
# まとめ
goroutineとは何かから始め、値のやり取りと実行フローを制御するためのchannel、そして、channel以外にもgoroutineの制御ができる同期オブジェクトとgoroutineを使用する時の注意点について話しました。いままでの内容を組み合わせてみると個人的にはgoroutineをchannelで値のやり取りしながら同期オブジェクトで非同期の問題点（race conditionなど）を制御し、contextでライフサイクルを管理するのがbest practiceかと思います。本記事を読まれ、goroutineの基礎と使い方について理解できたら良いと思います。






















