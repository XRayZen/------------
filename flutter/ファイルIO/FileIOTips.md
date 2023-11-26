# iOS の Files に表示させる
- Files アプリの中には「この iPhone 内」というな名前のディレクトリがあるけど、ここに書き込んだデータを表示させたいときはどうすればいいのだろう？
## info.plist に追記する
- この 2 つを追記すればよい！
>info.plist
```xml
<!-- on my iPhone (このiPhone内) に表示させるために必要  -->
    <key>LSSupportsOpeningDocumentsInPlace</key>
    <true/>
    <key>UIFileSharingEnabled</key>
    <true/>
<!-- //////////////////////////////////////////// -->
```
# 外部ライブラリでWeb/App対応のファイル保存
>https://pub.dev/packages/file_saver
- ただ、2023年1月12日時点ではバグ修正待ち
- Webではダウンロード先を選択できるが、iOS, Androidではアプリ内のフォルダに強制的に保存されるので注意。
- Webアプリの場合、debugだと直接ダウンロードフォルダに保存されるが、リリースするとダウンロード先を選択できた。
## ファイルを保存するコード
```dart
await FileSaver.instance.saveFile(String name, Uint8List bytes, String ext, mimeType: MimeType);
```
- nameはファイル名、bytesは実際のファイル（Uint8List形式）、extは拡張子を入力する。
- mimeTypeの指定は必須ではないが、Webで使うなら指定したほうがいいっぽい
  - （指定しないと拡張子が表示されない？かも。ちゃんと検証していないので確証はない）。
  - 指定できるMimeTypeは以下のページを参照。
  - ><https://pub.dev/documentation/file_saver/latest/file_saver/MimeType.html>
## 実際に使ったコード
>CSVを保存するコード。
```dart
String csv = const ListToCsvConverter().convert(_qaList);
Uint8List bytes = const Utf8Encoder().convert(csv);
await FileSaver.instance.saveFile(widget.title, bytes, 'csv', mimeType: MimeType.CSV);
```
- ```ListToCsvConverter().convert```は以下のcsvのパッケージを使ったやつ。
><https://pub.dev/packages/csv>
- ```Uint8List```を使うには、```import 'dart:typed_data';```が必要。
- ```Utf8Encoder().convert```を使うには、```import 'dart:convert';```が必要。
- ```widget.title```となっているやつは、遷移前の画面から持ってきたString値。
## 保存先を示す
Webでは保存先を選択できるが、iOSやAndroidでは保存先を選択できないため、ユーザーに保存先を表示したい。
- そこで、Snackbarを使って保存先を表示するよう、コードを以下のように修正した。
```dart
String csv = const ListToCsvConverter().convert(_qaList);
Uint8List bytes = const Utf8Encoder().convert(csv);
String path = await FileSaver.instance
    .saveFile(widget.title, bytes, 'csv', mimeType: MimeType.CSV);
if (!kIsWeb) {
  ScaffoldMessenger.of(context).showSnackBar(SnackBar(
    content: Text('$pathに保存されました'),
  ));
}
```


