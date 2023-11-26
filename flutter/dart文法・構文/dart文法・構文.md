
- dartはコードブロックの外でもグローバルインスタンスとして宣言できる
## 変数
変数宣言の方法
1. final
   - 定数として扱われ、再代入禁止
   - 型を特定させなければ、型推論（var）と同じ
2. const
   - コンパイルで評価された定数として扱われ、再代入禁止
   - 型を特定させなければ、型推論（var）と同じ
- どちらとも定数として扱われるが、constはコンパイルで評価され、finalは評価されない。
- 宣言の際に、すでに宣言された変数を代入すると、finalはコンパイルで評価しないためエラーにはならないが、constではエラーになる
## アクセシビリティ
- `_`を名前の先頭につけるとプライベート
## 遅延修飾子
- 変数の初期化のタイミングを遅らせるため
```dart
late String _temperature;
void heat() { _temperature = 'hot'; }
void chill() { _temperature = 'iced'; }
```
  - 例ではString型の_temperature変数の初期化をheat(),chill()メソッドが呼ばれるまで延期している
- 初期化と同時に使う
## コレクション
- コレクションに後で追加する用
```dart
var temps = [];//指定せずに宣言 List<Dynamic>
var hoges =<型>[];//事前に指定するこの方が良い
final List<String> tags;//クラスのメンバ用
//真にしないとリスト追加が出来ない
List.empty(growable=true);
//辞書型
Map<String, String> frameworks = {
 'Flutter' : 'Dart',
 'Rails' : 'Ruby',
};
```
## パターンマッチング
```dart
switch(enumType)//入れた後はコードアクション
```
## 関数による処理受け渡し
```dart
Future<void> Function(String message) noticeError;//関数定義
await noticeError(res.apiResponse.toString());//送信
Future<void> noticeError(String msg) async {}//受信
```
## 正しいNullCheck/型チェック
>dart型の解説:
>https://qiita.com/kabochapo/items/925a2cee9199031272df
```
if (nullかもしれないオブジェクト is 期待される型)
```
- もし == を使ってしまうと、間違った比較方法
- 「Equality operator == invocation with references of unrelated types.」と警告
## クラス/コンストラクタ
Dart のコンストラクタは二種類に分類
1. Generative コンストラクタ (生成的コンストラクタ)
   1. 普通のコンストラクタです。それを呼び出すことで、新しいインスタンスを返します。
2. Factory コンストラクタ
   1. 必ずしも新しいインスタンスを返す必要のないコンストラクタで、factory というキーワードをつけて作ります。
   2. ファクトリコンストラクタの典型的な使い方としては、シングルトンを作成する場合につかいます
   ```dart
    factory FeedModel.from(WebSite site) {
      return FeedModel(
        key: site.key,
        name: site.name,
        url: site.url,
        newCount: site.newCount,
        category: site.category,
        icon: site.icon,
        );
    }
    ```
>コンストラクタ。まずは基本
>コンストラクタ参考
>https://qiita.com/otome0927@github/items/e18e713db1705171d68e
>https://tech.excite.co.jp/entry/2022/09/12/120000
>https://flutter.keicode.com/dart/constructor.php
```dart
class Person {
  String firstName;
  String lastName;
  int age;

  Person(String firstName, String lastName, int age) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;
  }
  //単純に値をセットするだけのコンストラクタ これが普通
  Person(this.firstName, this.lastName, this.age);
  // Named constructor
  Person.namedConstructor(){
    //メンバに代入
  }

  sayHello() {
    print('Hi, I\'m $firstName $lastName ($age).');
  }
}

void main() {
  var p = new Person('Hanako', 'Yamada', 25);
  p.sayHello();
  // Hi, I'm Hanako Yamada (25).
}
```
- Dart のクラス定義は Java などによく似ています。次のように地味に必要なパラメータを全て渡して、 それぞれのインスタンス変数に代入することで、それらを初期化することができます。
### 現在の型を得る
- ランタイムの型を取得するには runtimeType を使います。
- Dart のあらゆるオブジェクトの基底クラスとなっている Object クラスが持つプロパティで、Type 型です。
```dart
final mario = Mario();
print(mario.runtimeType);    // Mario
```






