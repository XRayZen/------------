## 関数の書き方
```typescript
// 引数・戻り値に型を定義する(省略しても可)
function f(x: number): string {
    return x.toLocaleString() + '円';
}
console.log(f(1000)); // "1,000円"
```
```typescript
// 省略可能な引数は?を使う
function f(x: number, y?: string = '円'): string {
    return x.toLocaleString() + y;
}
console.log(f(1000)); // "1,000円"
console.log(f(1000, 'ペソ')); // "1,000ペソ"
// = 値 で代入すると値が指定されなかった時のデフォルト値を指定できる
function helloB(word = "world"): string {
  return "Hello, " + word;
}
```
## アロー関数式
- アロー関数式はfunctionを手軽に書くための1つの方法です。関数を値として渡す（コールバックやイベントリスナーなど）頻度の高いJavaScriptでは、かなり便利な機能です。
- ただ単に短く書けるだけでなく、thisの値を変更しないため、クラスの中で関数を作りたい場合、通常の関数よりアロー関数式の方が適切に働く場合が圧倒的に多いでしょう
```typescript
var add = (x: number): number => x + 1;
// {}を使ってもOK
var add = (x: number): number => {
    return x + 1;
}
```
## オーバーロード
- TypeScriptにもオーバーロードはありますが、JavaScriptコード生成の都合上、引数の型ごとに実装を持たせることはできません。
- そのため実装を与える宣言は、その他のオーバーロードの宣言のどのパターンでも対応できる形にする必要があります。
- このため、通常の実装時に利用することは少なく、型定義ファイルの作成時に利用する場合が大半です。
- 関数だけでなく、メソッドやコンストラクタなどでもオーバーロードを利用できます。
```typescript
function hello(value: number): string;
function hello(value: string): string;
function hello(value: any): string {
  if (typeof value === "number") {
    return new Array(value + 1).join("Hello!");
  } else if (typeof value === "string") {
    return "Hello, " + value;
  } else {
    return "Hello, unknown!";
  }
}

window.alert(hello(2));       // Hello!Hello! と表示される
window.alert(hello("world")); // Hello, world と表示される
```




