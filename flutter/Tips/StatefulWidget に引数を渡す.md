# StatefulWidget に引数(Parameter)を渡すには
!!! info riverpodでは使えない
>https://qiita.com/keiy121/items/72656a932256ec43d893
- コンストラクタによって引数を渡さず、
Stateオブジェクトに用意されているwidgetプロパティを利用して引数を取得する

```dart
class Page extends StatefulWidget {

  final String param; //上位Widgetから受け取りたいデータ
  Page({this.param}); //コンストラクタ

  @override
  _PageState createState() => _PageState();
}

class _PageState extends State<Page> {
  @override
  Widget build(BuildContext context) {
    return Text(widget.param); // widget 変数からデータ取得可能
  }
}
```
## info
「コンストラクタを用いてはいけない」などの記載はありませんが、下記の理由からwidgetプロパティを使用すべきと考えます

・Flutterの機能としてwidgetプロパティが用意され、使用が想定されている（公式ドキュメントやTutorial動画全てこの方式）
・二重にParameterの記載をすると実装ミス・修正影響が増えるという設計上のリスクがある。



