## 各端末固有のShareボタンを表示する
>https://pub.dev/packages/share_plus
1. `flutter pub add share_plus`


## iPadではこれが必要
```dart
// Shareボタンを呼び出すWidgetはビルダーでラップしてコンテキストを渡す必要がある
Builder(
  builder: (BuildContext context) {
    return ElevatedButton(
      onPressed: () => _onShare(context),
          child: const Text('Share'),
     );
  },
),
```
- 呼び出す関数
```dart
///シェア機能を呼び出す
///
///iPadの場合は呼び出すWidgetをビルダーでラップする必要がある
Future<void> callShare(BuildContext context, String url, String subject) async {
  final box = context.findRenderObject() as RenderBox?;
  //端末固有のシェア機能を呼ぶ
  await Share.share(
    url,
    subject: subject,
    sharePositionOrigin: box!.localToGlobal(Offset.zero) & box.size,
  );
}
```