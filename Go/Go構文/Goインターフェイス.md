# インターフェイス
## インターフェースを定義する
インターフェースを定義するときは、次のように interface キーワードを使用します。
```go
type インターフェイス型名 interface {
	メソッド名①(引数名 引数の型) 戻り値の型
	メソッド名②(引数名 引数の型) 戻り値の型
	︙
}
```
### インターフェイスを関数に実装する
```go
func 関数名(引数名 インターフェイス型名) 戻り値の型 {
    ︙
}
```
上記のように インターフェイス型を設定することで、 この関数は「 XXX と AAA メソッドを実装したインターフェイス型のみ引数として受け入れる」と示すことができます。
## 例として
例えば、Vehicle 型の場合は次のように定義します。
```go
type Vehicle interface {
	Accelerate()
	Brake()
}
```
このインターフェースでは、Accelerate（引数なし・戻り値なし）と Brake（引数なし・戻り値なし）の2つのメソッドが必要であることを示しています。

では、この Vehicle 型を drive 関数の引数の型に設定してみましょう。
```go
func drive(vehicle Vehicle) {
	vehicle.Accelerate()
	vehicle.Brake()
}
```
上記のように Vehicle 型を設定することで、 drive 関数は「 Accelerate と Brake メソッドを実装した型のみ引数として受け入れる」と示すことができます。

実際に、これまで定義した Car 型と Bike 型を振り返って見てみましょう。

>Car型
```go
type Car struct {
	color string
}

func (c Car) Accelerate() {
	fmt.Println("車が加速する")
}

func (c Car) Brake() {
	fmt.Println("車がブレーキをかける")
}

<!-- Bike型 -->
type Bike struct {
	color string
}

func (b Bike) Accelerate() {
	fmt.Println("バイクが加速する")
}

func (b Bike) Brake() {
	fmt.Println("バイクがブレーキをかける")
}
```
それぞれの構造体には、Accelerate と Brake メソッドが定義されています。この2つのメソッドは、Vehicle 型で宣言されたメソッドと名前・引数・戻り値の型すべて一致しているため、これらの構造体は Vehicle インターフェースを実装している（満たしている）と見なされます。

インターフェースを実装していると見なされたら、そのインターフェース型で宣言された変数や関数の引数には、どんな値でも入れることができるようになります。
```go
type Vehicle interface {
	Accelerate()
	Brake()
}

type Car struct {
	color string
}

func (c Car) Accelerate() {
	fmt.Println("車が加速する")
}

func (c Car) Brake() {
	fmt.Println("車がブレーキをかける")
}

type Bike struct {
	color string
}

func (c Bike) Accelerate() {
	fmt.Println("バイクが加速する")
}

func (c Bike) Brake() {
	fmt.Println("バイクがブレーキをかける")
}

func drive(vehicle Vehicle) {
	vehicle.Accelerate()
	vehicle.Brake()
}

func main() {
	var car Car = Car{}
	drive(car)

	var bike Bike = Bike{}
	drive(bike)
}
```
>実行結果
```bash
車が加速する
車がブレーキをかける
バイクが加速する
バイクがブレーキをかける
```





































