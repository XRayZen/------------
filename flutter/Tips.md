
- ProtoBufのAPIリポジトリ層にアーキテクチャ的にドメイン層オブジェクトと相互変換する必要がある
  - Int64からIntに変換するにはfixumをインポートする必要
  - DateTimeからmilisecoundをStringにしてint64.parseでInt64に変換して渡す
  - 受け取る時はInt64をIntにしてDateTime.milisecoundでパースする

- ユニットテストだとAPI越しの重い処理はタイムアウトする
# プロジェクトフォルダの移動
移動したらVsCodeを開き直して`build`フォルダを削除する
# Record型(タプル)有効
```command
flutter pub add tuple
```
```dart
const t = Tuple2<String, int>('a', 10);
print(t.item1); // prints 'a'
print(t.item2); // prints '10'
```
## プラットフォーム判定
```dart
import 'dart:io';

!kIsWeb && Platform.isAndroid
//動作しているプラットフォームを判定する
Platform.isIOS //iOS
Platform.isAndroid //Android
Platform.isMacOS //MacOS
Platform.isWindows //Windows
Platform.isLinux //Linux
Platform.isFuchsia //Fuchsia
```
## 背景
- webだとdart:ioを使えない
- universal_platformを使えば`UniversalPlatform.isAndroid`のようにプラットフォームを判別できる
- `flutter pub add universal_platform`
>pubspec.yaml
```yaml
universal_platform: ^1.0.0+1
```
>https://zenn.dev/ryo_ryukalice/articles/140a64f894afad

# Flutterでテキスト内のURLをチェックする方法
>https://qiita.com/HisakoIsaka/items/b0e58afa3593032bedfb
```dart
///テキスト内にURLが含まれていたら分割して返す<br/>
///無いならnull
List<String>? parseUrls(String word) {
  final urlRegExp = RegExp(
    r'((https?:www\.)|(https?:\/\/)|(www\.))[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9]{1,6}(\/[-a-zA-Z0-9()@:%_\+.~#?&\/=]*)?',
  );
  // ここで、Iterable型でURLの配列を取得
  final urlMatches = urlRegExp.allMatches(word);
  final urls = urlMatches
      .map((urlMatch) => word.substring(urlMatch.start, urlMatch.end))
      .toList();
  if (urls.isEmpty) {
    return null;
  } else {
    return urls;
  }
}
```

