# テキスト関連
## レスポンシブ対応のテキストスタイルを生成
- `DefaultTextStyle.merge`のスタイルをセットする時に使う
```dart
///レスポンシブ対応のテキストスタイルを生成
TextStyle genResponsiveTextStyle(
    BuildContext context,
    double mobileValue,
    double tabletValue,
    double? letterSpacing,
    FontWeight? fontWeight,
    Color? color) {
  final value = ResponsiveValue(
    context,
    defaultValue: 25.0,
    valueWhen: [
      //MOBILE より小さい場合はフォントサイズがvalue  になる
      Condition.smallerThan(name: MOBILE, value: mobileValue),
      // TABLET より大きい場合はフォントサイズが value になる
      Condition.largerThan(name: TABLET, value: tabletValue)
    ],
  ).value;
  return TextStyle(
      color: color,
      fontSize: value,
      letterSpacing: letterSpacing,
      fontWeight: fontWeight);
}
```
## 指定したWidget範囲のTextStyleを一括変更する
- Widgetの中にあるTextだけに共通したTextStyleにしたい時に使う
- DefaultTextStyle.mergeでWidgetを囲い、styleを指定すればWidget Tree内のすべてのTextにstyleが適用される
```dart
child: DefaultTextStyle.merge(
  style: const TextStyle(color: Colors.red, fontSize: 20),
  child: Column(
    children: const [
      Text('ホーム'),
      Text('事業内容'),
      Text('製品情報'),
      Text('お問い合わせ'),
      Text('会社概要'),
    ],
  ),
),
```

