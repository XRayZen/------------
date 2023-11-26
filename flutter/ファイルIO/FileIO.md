- Webは当然非対応
## ディレクトリの作成
```
final directory = Directory('作りたいディレクトリの path');
await directory.create(recursive: true);
```
## ファイルの読み込み
```
final file = File('読み込みたいファイルの path');
final String content = await file.readAsString(); // テキストデータならこう書く
```
## ファイルの書き込み
```
final file = File('書き込みたいファイルの path');
await file.writeAsString(DateTime.now().toString()); // テキストデータならこう書く
```
## テキストとバイナリの相互変換

1. Convert String to Uint8List
```dart
Uint8List convertStrToBytes(String str) {
  final List<int> codeUnits = str.codeUnits;
  final Uint8List unit8List = Uint8List.fromList(codeUnits);

  return unit8List;
}
```
2. Convert Uint8List to String
```dart
String convertBytesToStr(Uint8List uint8list) {
  return String.fromCharCodes(uint8list);
}
```

この[FileSystemEntity]を削除します。

FileSystemEntity]がディレクトリの場合、[recursive]がfalseなら、ディレクトリは空でなければなりません。それ以外の場合、[recursive] が true ならば、ディレクトリとその中のすべてのサブディレクトリとファイルが削除されます。再帰的に削除する場合、リンクは追跡されません。リンクだけが削除され、そのターゲットは削除されません。

recursive] が true の場合、[FileSystemEntity] のタイプがファイル システムのコンテンツと一致しない場合でも、[FileSystemEntity] は削除されます。この動作により、[delete] を使用して任意のファイル システム オブジェクトを無条件に削除することができます。

削除が完了すると、この [FileSystemEntity] で完了する Future<FileSystemEntity> を返します。FileSystemEntity] を削除できない場合、例外が発生して未来が完了します。



















