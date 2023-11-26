>https://pub.dev/packages/hive
# BoxCollections 
BoxCollectionsは、通常のボックスと同様に使用できるボックスの集合体ですが、Web上での動作速度を飛躍的に向上させることができます。コレクション内のすべてのボックスを一度に開いたり閉じたりすることができ、ウェブ上のインデックス付きDBにデータをより効率的に保存することができます。

また、Web上の膨大な数のデータベーストランザクションを高速化するために使用できるTransactionsも公開されています。

dart:ioプラットフォームでは、BoxCollectionsやTransactionsによる性能向上はありません。BoxCollectionsだけは、ボックスの階層化や開発経験を積むのに有効かもしれません。
```dart
// Create a box collection
  final collection = await BoxCollection.open(
    'MyFirstFluffyBox', // データベースの名前
    {'cats', 'dogs'}, // 箱の名前
    path: './', // ボックスを格納するパス (Flutter / Dart IOでのみ使用)
    key: HiveCipher(), // ボックスを暗号化するためのキー（Flutter / Dart IOでのみ使用できる）
  );

  //Open your boxes. 
  //Optional: Give it a type.
  final catsBox = collection.openBox<Map>('cats');

  // Put something in
  await catsBox.put('fluffy', {'name': 'Fluffy', 'age': 4});
  await catsBox.put('loki', {'name': 'Loki', 'age': 2});

  // Get values of type (immutable) Map?
  final loki = await catsBox.get('loki');
  print('Loki is ${loki?['age']} years old.');

  // Returns a List of values
  final cats = await catsBox.getAll(['loki', 'fluffy']);
  print(cats);

  // Returns a List<String> of all keys
  final allCatKeys = await catsBox.getAllKeys();
  print(allCatKeys);

  // Returns a Map<String, Map> with all keys and entries
  final catMap = await catsBox.getAllValues();
  print(catMap);

  // delete one or more entries
  await catsBox.delete('loki');
  await catsBox.deleteAll(['loki', 'fluffy']);

  // ...or clear the whole box at once
  await catsBox.clear();

  // トランザクションによる書き込み動作の高速化
  await collection.transaction(
    () async {
      await catsBox.put('fluffy', {'name': 'Fluffy', 'age': 4});
      await catsBox.put('loki', {'name': 'Loki', 'age': 2});
      // ...
    },
    boxNames: ['cats'], // By default all boxes become blocked.
    readOnly: false,
  );
```
















































