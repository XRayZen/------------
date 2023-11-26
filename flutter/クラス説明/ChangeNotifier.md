## ChangeNotifierとは？
- 今はriverpodがあるため非推奨
  - ChangeNotifier は、リスナーに対して変更通知を行うクラスです。
  - 公式ドキュメント
```
通知用に VoidCallback を使用した変更通知 API を提供する、拡張や混在が可能なクラスです。
```
- リスナーの追加にはO(1)、リスナーの削除と通知の発信にはO(N)である（Nはリスナーの数）。
- Flutterでchange notifierを利用するにはいくつかの方法があります。
  - ChangeNotifierの.addListenerメソッドを使用する。
  - ChangeNotifierProvider、Consumer、Providerの組み合わせを使用します。
- これらの機能はすべて、Provider パッケージによって提供されています。
### 2つ目のアプローチ
現実世界では、他のクラスがChangeNotifierオブジェクトをリッスンすることができます。ChangeNotifierが更新された値を取得すると、notifyListeners と呼ばれるメソッドを呼び出し、そのリスナーのいずれかが更新された値を受け取ることができます。
```dart
class Person extends ChangeNotifier {
  Person({this.name, this.age});
  final String name;
  int age;
         
  void increaseAge() {
    this.age++;
    notifyListeners();//UIに変更を通知
  }
}
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => Person(name: "Joe", age: 28),
      child: MyApp(),
    ),
  );
}
```
アプリ内部では、このPersonをリスニングしているクラスには、年齢が変更された場合に通知が行われます。内部的には、notifyListenersは登録されたリスナーを呼び出します。