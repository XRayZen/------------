# json相互変換
## コードジェネレーターライブラリを使う
```yaml
dependencies:
  json_annotation: ^4.7.0

dev_dependencies:
  build_runner: ^1.0.0
  json_serializable: ^6.5.4
```
- Userクラスを定義してJsonSerializableアノテーションを付与
>user.dart
```dart

import 'package:json_annotation/json_annotation.dart';

// ジェネレートされたクラスからUserクラスのprivateメンバ変数にアクセスするため
part 'user.g.dart';//importではなくpart

@JsonSerializable()
class User {
  User(this.name, this.email);

  String name;
  String email;

  //生成されないので自分で書く！
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);

  //生成されないので自分で書く！2023年1月12日
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```
## Unit8が定義クラスに含まれている場合
- Unit8Listを変換するクラスを追加する
- JsonConvertを承継する
```dart
import 'dart:typed_data';
import 'package:json_annotation/json_annotation.dart';

class Uint8ListConverter implements JsonConverter<Uint8List, List<int>> {
  const Uint8ListConverter();

  @override
  Uint8List fromJson(List<int> json) {
    if (json.isEmpty) {
      return Uint8List(0);
    }

    return Uint8List.fromList(json);
  }

  @override
  List<int> toJson(Uint8List object) {
    if (object.isEmpty) {
      return Uint8List(0);
    }

    return object.toList();
  }
}
```
- 定義クラスのUnit8上に書いておく
```dart
@Uint8ListConverter()
Uint8List profile = null;
```
- コマンドを実行
```PowerShell
flutter packages pub run build_runner build
```
- もしくは
```PowerShell
flutter packages pub run build_runner build --delete-conflicting-outputs
```
## 変換
```dart
var entityJson = await compute(jsonDecode, dataEntity);
var entity = ModelImageMeta.fromJson(entityJson).to(imageData);
```
```dart
var json = jsonEncode(ModelImageMeta.from(entity).toJson());
```









