# Notifier
>https://zenn.dev/10_tofu_01/articles/try_riverpod_generator
- Riverpod 2.0 新要素の Notifier
- Notifier は Riverpod 2.0 で新しく追加されたもので、StateProvider と StateNotifierProvider の両方の挙動をカバーできるようなプロバイダ
- これまでは単純な数値の加算減算や、Enum の切り替えなどをする分には StateProvider を用いて、複雑な実装をしたい場合には StateNotifier を使うという運用だったが、今後は Notifier（と AsyncNotifier） 推奨となる
## 使い方
Notifier を継承したクラスを作成することで、StateNotifier のようにより柔軟な State の監視が可能に
```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  // オプション
  void increment() => state++;
}
```
- build メソッドをオーバーライドして初期値を StateProvider のように設定し、必要に応じて StateNotifier のように関数を定義
### プロバイダー作成
どちらの書き方でも結構
```dart
final counterProvider = NotifierProvider<Counter, int>(() => Counter());
// or
final counterProvider = NotifierProvider<Counter, int>(Counter.new);
```
## コード生成
riverpod_generator の自動生成の対象となっています。
書き方も基本的には記事の最初に書いた方法に倣うだけで、気をつける部分は Notifier<int> ではなく _$Counter を継承しているところでしょうか。 Freezed や Hive などで似たような書き方をします
```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
part 'counter_provider.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}
```
自動生成を使うと生成ファイル内でプロバイダを作成してくれる


