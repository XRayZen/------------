# Provider
- 公式Doc
- https://zuma-lab.com/posts/flutter-search-github-riverpod-di
## 概要
- プロバイダの中で最もベーシックなプロバイダであり、値を同期的に生成してくれる
- Provider は`ref.watch`と組み合わせることで、同期的な処理の結果をキャッシュする
- `ref.watch`があるおかげで、`Provider` は値を再評価するタイミングを自動検知することができる
  - パフォーマンスが改善し、さらにロジックとウィジェットを分離して管理することができるようになる
  - `Provider`の値が再評価された際に、値が以前の値と変わらない場合はそのプロバイダもしくはウィジェットは更新されない
##### 一般的には次のような用途で使われます。
- 計算結果をキャッシュする
- 他のプロバイダに値（例えば`Repository`含むビジネスロジック層や`HttpClient`含むインフラ層などのインスタンス）を公開する
- DIをするために`Impl`インスタンスを収容する
  ```dart
  final apiClientProvider = Provider.autoDispose(
    (_) => GithubApiClientImpl(),
  );
  ```
- テスト実施時やウィジェット構築時に値をオーバーライドするため。
- `select`を使わずにプロバイダやウィジェットの更新の条件を限定する
### Provider を使って計算結果をキャッシュする
- Provider は`ref.watch`と組み合わせることで、同期的な処理の結果をキャッシュする

!!! example 具体例
    例えば、Todo リストにフィルタを適用するような場合です。 
    この種のフィルタリングは負荷が若干高くなる可能性があるため、画面が再描画するたびにフィルタが適用されてしまうようなことは避けたいものです。 
    そこで登場するのが Provider です。Provider にフィルタリングを任せて、その計算結果をキャッシュしましょう。
- Todo リストを管理する、次のような StateNotifierProvider があるとします。
```java
class Todo {
  Todo(this.description, this.isCompleted);
  final bool isCompleted;
  final String description;
}

class TodosNotifier extends StateNotifier<List<Todo>> {
  TodosNotifier() : super([]);

  void addTodo(Todo todo) {
    state = [...state, todo];
  }
  //TODO "removeTodo" のような他のメソッドを追加する
}

final todosProvider = StateNotifierProvider<TodosNotifier, List<Todo>>((ref) {
  return TodosNotifier();
});
```
そして Provider を使って Todo リストをフィルタリングし、完了タスクのみを残してリストを公開します。
>Porvider定義
```dart
final completedTodosProvider = Provider<List<Todo>>((ref) {
  // todosProvider から Todo リストの内容をすべて取得
  final todos = ref.watch(todosProvider);

  // 完了タスクのみをリストにして値として返す
  return todos.where((todo) => todo.isCompleted).toList();
});
```
ウィジェットは`completedTodosProvider`を監視(`ref.watch`)することで、完了タスクのみを UI として表示することができます。
>Widgetでの使用
```dart
Consumer(builder: (context, ref, child) {
  final completedTodos = ref.watch(completedTodosProvider);
  // TODO ListView/GridView/... を使って Todo リストを表示する 
});
```
- これでフィルタリングの計算結果をキャッシュすることができました。
- `completedTodosProvider` が公開する完了タスクのリストは、何度値を取得しようが、新たに Todo が追加・削除・更新されるまで再評価されることはありません。
- また、Todo リストの内容が変わった際に手動でキャッシュを無効化する必要がない点にお気づきでしょうか。 
- `ref.watch`があるおかげで、`Provider` は値を再評価するタイミングを自動検知することができる
## 他のプロバイダと異なる点
- プロバイダやウィジェットの更新の条件を限定出来る
- ref.watch を使うなどして Provider の値が再評価された際に、値が以前の値と変わらない場合はそのプロバイダもしくはウィジェットは更新されない

この特性を活用できる例としては、ページネーションが施されたページの「戻る/次へ」ボタンの有効・無効の切り替えが挙げられる
![](images/47580830-31263a00-d950-11e8-9b61-0eaddab2709e.png)
ここでは「戻る」ボタンにのみフォーカスしてサンプルコードをご紹介したいと思います。 まず、現在のページインデックスを取得し、それが 0 に等しい場合はボタンが無効化するようなウィジェットを準備します。

コードは次のようになります。
```dart
final pageIndexProvider = StateProvider<int>((ref) => 0);

class PreviousButton extends ConsumerWidget {
  const PreviousButton({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 現在のページが最初のページ（0）でない場合、戻るボタンを有効にする
    final canGoToPreviousPage = ref.watch(pageIndexProvider) != 0;

    void goToPreviousPage() {
      ref.read(pageIndexProvider.notifier).update((state) => state - 1);
    }

    return ElevatedButton(
      onPressed: canGoToPreviousPage ? null : goToPreviousPage,
      child: const Text('previous'),
    );
  }
}
```
しかし、このコードには問題があります。それは現在のページが変わるたびに「戻る」ボタンが更新されてしまうことです。 ここはボタンの有効・無効が切り替わるときにのみ、ボタンが更新されるのが理想的ですよね。

この問題の要因は、ユーザが前のページに戻れるか否かの指標を「戻る」ボタンのウィジェットが直接計算してしまっていることにあります。

よって、この問題を解決するにはロジックをウィジェットから抽出して Provider に計算を任せる必要があります。
```dart
final pageIndexProvider = StateProvider<int>((ref) => 0);

// ユーザが前のページに戻れるかどうかを計算するプロバイダ
final canGoToPreviousPageProvider = Provider<bool>((ref) {
  return ref.watch(pageIndexProvider) != 0;
});

class PreviousButton extends ConsumerWidget {
  const PreviousButton({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 新しく作成したプロバイダを監視
    // ウィジェットはこれでもう前のページに戻れるかの計算をする必要がない
    final canGoToPreviousPage = ref.watch(canGoToPreviousPageProvider);

    void goToPreviousPage() {
      ref.read(pageIndexProvider.notifier).update((state) => state - 1);
    }

    return ElevatedButton(
      onPressed: canGoToPreviousPage ? null : goToPreviousPage,
      child: const Text('previous'),
    );
  }
}
```
このような小さなリファクタリングを行うことで、PreviousButton ウィジェットはページインデックスが変わっても更新されることがなくなりました。

ページインデックスが変わると canGoToPreviousPageProvider の値は再評価されますが、 値が直前に公開したものと変わらなければ、PreviousButton は更新されません。

これでボタンのパフォーマンスが改善し、さらにロジックとウィジェットを分離して管理することができるようになりました。 これは Provider のユニークな特性のおかげによるものです。









