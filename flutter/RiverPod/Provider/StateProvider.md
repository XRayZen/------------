# StateProvider
- StateProvider は外部から変更が可能なステート（状態）を公開するプロバイダ
- `StateNotifierProvider` の簡易版であり、ステートの管理にわざわざ`StateNotifier` クラスを定義するほどではない場合にご利用いただけます。
- StateProvider は UI 側で利用される シンプルなステート を管理するのにうってつけ
## やり方
### 定義
```dart
final isFeedsEditModeProvider = StateProvider<FeedsEditMode>((ref) {
  return FeedsEditMode.noEdit;
});
```
### 利用方法
```dart
var isEditMode = ref.watch(isFeedsEditModeProvider);
isEditMode =
 isEditMode == FeedsEditMode.edit
                ? FeedsEditMode.noEdit
                : FeedsEditMode.edit;
```
### 更新
```dart
ref.read(isFeedsEditModeProvider.notifier).state=isEditMode;
//updateは今の値を加工した結果を設定するときに使う
ref.read(_counterProvider.notifier).update((state) => state + 1);
```
## シンプルなステートとは、次のような型のステートのことを指します。
- 列挙型（enum）、例えばフィルタの種類など
- 文字列型、例えばテキストフィールドの入力内容など
- bool 型、例えばチェックボックスの値など
- 数値型、例えばページネーションのページ数やフォームの年齢など
## 次のようなステートを公開するために使うべきではありません。
- ステートの算出に何かしらのバリデーション（検証）ロジックが必要
- ステート自体が複雑なオブジェクトである（カスタムのクラスや List/Map など）
- ステートを変更するためのロジックが単純な count++ よりは高度である必要がある

上記のように一歩踏み込んだステート管理が必要な場合は、カスタムの StateNotifier クラスを定義した上で StateNotifierProvider を利用することをおすすめします。
この場合は最初に多少のボイラープレートコードの設定が必要になりますが、長期的な観点からプロジェクトの保守性を考えれば、 ステートのビジネスロジックを一箇所で集中管理できるため得策だと言えます。
## 使用例: ドロップダウンメニューを使ってフィルタの種類を切り替える
StateProvider の代表的なユースケースとしては、ドロップダウンメニューやテキストフィールド、 チェックボックスなどフォームに使用されるコンポーネントのステート管理が挙げられます。 ここでは商品リストのソートの種類を切り替えられるドロップダウンメニューを、StateProvider を利用して実装していきたいと思います。

商品（Product）の定義と商品リストの内容は次の通りです。
```dart
class Product {
  Product({required this.name, required this.price});

  final String name;
  final double price;
}

final _products = [
  Product(name: 'iPhone', price: 999),
  Product(name: 'cookie', price: 2),
  Product(name: 'ps5', price: 500),
];

final productsProvider = Provider<List<Product>>((ref) {
  return _products;
});
```
実際のアプリでは商品リストは FutureProvider のネットワークリクエストを通じて取得するものかと思いますが、ここでは簡略化のためハードコーディングしています。

そして UI 側は次のように商品リストを表示します。
```dart
Widget build(BuildContext context, WidgetRef ref) {
  final products = ref.watch(productsProvider);
  return Scaffold(
    body: ListView.builder(
      itemCount: products.length,
      itemBuilder: (context, index) {
        final product = products[index];
        return ListTile(
          title: Text(product.name),
          subtitle: Text('${product.price} \$'),
        );
      },
    ),
  );
}
```
土台ができたところで、ここにドロップダウンメニューを追加しましょう。 このドロップダウンメニューは商品リストを値段順か名前順かでソートしてくれます。 メニューを表示するボタンには Flutter の DropDownButton を使います。
```dart
// フィルタの種類を表す列挙型
enum ProductSortType {
  name,
  price,
}

Widget build(BuildContext context, WidgetRef ref) {
  final products = ref.watch(productsProvider);
  return Scaffold(
    appBar: AppBar(
      title: const Text('Products'),
      actions: [
        DropdownButton<ProductSortType>(
          value: ProductSortType.price,
          onChanged: (value) {},
          items: const [
            DropdownMenuItem(
              value: ProductSortType.name,
              child: Icon(Icons.sort_by_alpha),
            ),
            DropdownMenuItem(
              value: ProductSortType.price,
              child: Icon(Icons.sort),
            ),
          ],
        ),
      ],
    ),
    body: ListView.builder(
      // ... 
    ),
  );
}
```
ドロップダウンメニューが完成したら、StateProvider を作成してプロバイダとドロップダウンメニューのステートを同期させましょう。

まずは StateProvider の作成から。
```dart
final productSortTypeProvider = StateProvider<ProductSortType>(
  // ソートの種類 name を返します。これがデフォルトのステートとなります。
  (ref) => ProductSortType.name,
);
```
そして、次のようにプロバイダとドロップダウンメニューを紐づけます。
```dart
DropdownButton<ProductSortType>(
  // ソートの種類が変わると、ドロップダウンメニューが更新されて
  // 表示されるアイコン（メニューアイテム）が変わります。
  value: ref.watch(productSortTypeProvider),
  // ユーザがドロップダウンメニューを操作するとプロバイダのステートが更新されます。
  onChanged: (value) =>
      ref.read(productSortTypeProvider.notifier).state = value!,
  items: [
    // ...
  ],
),
```
これによりメニューからソートの種類を選ぶことが可能になりました。 ただし、これだけでは商品リストはソートされません! 最後に productsProvider を更新することで実際のソートを行う必要があります。

この機能を実装するカギとなるのは ref.watch です。 ref.watch を使って productsProvider に現在のソートの種類を監視・取得させ、 その値が変わるたびに商品リストを再評価させます。

具体的な実装は次の通りです。
```dart
final productsProvider = Provider<List<Product>>((ref) {
  final sortType = ref.watch(productSortTypeProvider);
  switch (sortType) {
    case ProductSortType.name:
      return _products.sorted((a, b) => a.name.compareTo(b.name));
    case ProductSortType.price:
      return _products.sorted((a, b) => a.price.compareTo(b.price));
  }
});
```
これだけです! この変更により UI はソートの種類が変わったことを検知して、自動的に商品リストを更新してくれます。


## update を使って直近のステートから新たなステートを算出する
StateProvider の直近のステートをもとに新たなステートを算出するようなケースでは、 以下のようなコードを書いてしまいがちです。
```dart
final counterProvider = StateProvider<int>((ref) => 0);

class HomeView extends ConsumerWidget {
  const HomeView({Key? key}): super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // 直近のステートから新たなステートを算出しようとすると、
          // このようにプロバイダを2回利用してしまいがち。
          ref.read(counterProvider.notifier).state = ref.read(counterProvider.notifier).state + 1;
        },
      ),
    );
  }
}
```

このコードは間違いではありませんが、ちょっと書くのが億劫ですよね。

もう少しコードを簡略化したい場合は update メソッドを使ってください。 このメソッドに、新しいステートを戻り値とするコールバック関数を渡します。 このコールバック関数は直近のステートの値をパラメータとして利用できます。 update メソッドを使ってコードをリファクタリングした例がこちらです。
```dart
final counterProvider = StateProvider<int>((ref) => 0);

class HomeView extends ConsumerWidget {
  const HomeView({Key? key}): super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ref.read(counterProvider.notifier).update((state) => state + 1);
        },
      ),
    );
  }
}
```
効果はそのままに、簡潔で見やすいコードにすることができました。




