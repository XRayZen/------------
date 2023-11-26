# FutureProvider
- FutureProvider は非同期操作が可能な Provider である

## 次のような用途で使われます。
- 非同期操作を実行し、その結果をキャッシュするため（例えばネットワークリクエストなど）。
- 非同期操作の error/loading ステートを適切に処理するため。
- 非同期的に取得した複数の値を組み合わせて一つの値にするため。
- FutureProvider は ref.watch と組み合わせることでその効果を発揮します。 
  - 例えば、何かの値が変わったときに自動でデータを再取得するよう設定することで、プロバイダが常に最新データを外部に公開することを保証できます。
>備考
  FutureProvider には値を計算する処理を直接変更する手段がありません（ユーザ操作時）。
  値を計算するメソッドが複数必要など、より高度なことをする場合は StateNotifierProvider の使用を検討してください。
## 定義
```dart
// FutureProviderの作成 (単一のデータを非同期で取得する)
final futureProvider = FutureProvider<dynamic>((ref) async {
  await Future.delayed(const Duration(seconds: 3));
  return 'Hello World';
});
```

## 使用例: 設定ファイルを取得する
FutureProvider は例えば、次のような用途に適しています:
- ファイル/データを外部から読み込み、そのデータをもとに Configuration オブジェクトを生成、プロバイダの値として外部へ公開する。

ファイル・データの読み込みはプロバイダ内で async/await を使うことで可能です。 
- Flutter のアセットからファイルを読み込む場合は、次のようになります。
```dart
@riverpod
Future<Configuration> fetchConfigration(FetchConfigrationRef ref) async {
  final content = json.decode(
    await rootBundle.loadString('assets/configurations.json'),
  ) as Map<String, Object?>;

  return Configuration.fromJson(content);
}
```
UI 側では以下の通り Configuration を監視します。
```dart
Widget build(BuildContext context, WidgetRef ref) {
  final config = ref.watch(fetchConfigrationProvider);

  return config.when(
    loading: () => const CircularProgressIndicator(),
    error: (err, stack) => Text('Error: $err'),
    data: (config) {
      return Text(config.host);
    },
  );
}
```
これにより Future が解決すると UI は自動で更新されます。 おまけにキャッシュ機能が働くため、複数のウィジェットがこの Configuration を必要とする場合でもアセットは一度しか読み込まれません。

そして、ご覧いただける通り FutureProvider を監視した際の戻り値は AsyncValue です。 AsyncValue のおかげで error/loading などのステートを適切にウィジェットに変換することができます。











