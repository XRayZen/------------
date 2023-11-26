## ローカルパスを取得
- 当然だがWebは非対応
>https://pub.dev/packages/path_provider
```yaml
path_provider: ^2.0.11
```
```dart
Directory tempDir = await getTemporaryDirectory();
String tempPath = tempDir.path;
Directory appDocDir = await getApplicationDocumentsDirectory();
String appDocPath = appDocDir.path;
```




