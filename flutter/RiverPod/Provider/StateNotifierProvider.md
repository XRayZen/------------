# StateNotifierProvider
- StateNotifierProvider は StateNotifier（Riverpod が依存する state_notifier パッケージのクラス）を監視し、公開するためのプロバイダです。 
- ユーザ操作などにより変化するステート（状態）を管理するソリューションとして Riverpod が推奨だが
- Riverpod 2.0 新要素の `Notifier`はStateProvider と StateNotifierProvider の両方の挙動をカバーできるので今後はそれを推奨
## 次のような用途で使われます。
- 「イミュータブル（不変）」 なステートを公開するため
  - （イミュータブルではあるが、イベントに応じて変わることがある）
- ステートを変更するためのロジック（いわゆるビジネスロジック）を一つの場所で集中管理して保守性を高めるため。

## ここで具体例として、
- StateNotifierProvider を使って Todo リストを管理します。 
- StateNotifierProvider を使うことで addTodo などのメソッドを公開し、UI 側から Todo リストの内容を操作できるようにします。
```dart
// StateNotifier のステート（状態）はイミュータブル（不変）である必要があります。
// ここは Freezed のようなパッケージを利用してイミュータブルにしても OK です。
@immutable
class Todo {
  const Todo({required this.id, required this.description, required this.completed});

  // イミュータブルなクラスのプロパティはすべて `final` にする必要があります。
  final String id;
  final String description;
  final bool completed;

  // Todo はイミュータブルであり、内容を直接変更できないためコピーを作る必要があります。
  // これはオブジェクトの各プロパティの内容をコピーして新たな Todo を返すメソッドです。
  Todo copyWith({String? id, String? description, bool? completed}) {
    return Todo(
      id: id ?? this.id,
      description: description ?? this.description,
      completed: completed ?? this.completed,
    );
  }
}

// StateNotifierProvider に渡すことになる StateNotifier クラスです。
// このクラスではステートを `state` プロパティの外に公開しません。
// つまり、ステートに関しては public なゲッターやプロパティは作らないということです。
// public メソッドを通じて UI 側にステートの操作を許可します。
class TodosNotifier extends StateNotifier<List<Todo>> {
  // Todo リストを空のリストとして初期化します。
  TodosNotifier(): super([]);

  // Todo の追加
  void addTodo(Todo todo) {
    // ステート自体もイミュータブルなため、`state.add(todo)`
    // のような操作はできません。
    // 代わりに、既存 Todo と新規 Todo を含む新しいリストを作成します。
    // Dart のスプレッド演算子を使うと便利ですよ!
    state = [...state, todo];
    // `notifyListeners` などのメソッドを呼ぶ必要はありません。
    // `state =` により必要なときに UI側 に通知が届き、ウィジェットが更新されます。
  }

  // Todo の削除
  void removeTodo(String todoId) {
    // しつこいですが、ステートはイミュータブルです。 
    // そのため既存リストを変更するのではなく、新しくリストを作成する必要があります。
    state = [
      for (final todo in state)
        if (todo.id != todoId) todo,
    ];
  }

  // Todo の完了ステータスの変更
  void toggle(String todoId) {
    state = [
      for (final todo in state)
        // ID がマッチした Todo のみ、完了ステータスを変更します。
        if (todo.id == todoId)
          // またまたしつこいですが、ステートはイミュータブルなので
          // 状態を変更するにはTodo クラスに実装した `copyWith` メソッドを使用して
          // Todo オブジェクトのコピーを作る必要があります。
          todo.copyWith(completed: !todo.completed)
        else
          // ID が一致しない Todo は変更しません。
          todo,
    ];
  }
}

// 最後に TodosNotifier のインスタンスを値に持つ StateNotifierProvider を作成し、
// UI 側から Todo リストを操作することを可能にします。
final todosProvider = StateNotifierProvider<TodosNotifier, List<Todo>>((ref) {
  return TodosNotifier();
});
```
- これで StateNotifierProvider の定義は完了です。
- 次にウィジェット内でプロバイダを利用し、Todo リストの内容表示と操作の部分を実装します。
```dart
class TodoListView extends ConsumerWidget {
  const TodoListView({Key? key}): super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Todo リストの内容に変化があるとウィジェットが更新される
    List<Todo> todos = ref.watch(todosProvider);

    // スクロール可能なリストビューで Todo リストの内容を表示
    return ListView(
      children: [
        for (final todo in todos)
          CheckboxListTile(
            value: todo.completed,
            // 各 Todo をタップすると、完了ステータスを変更できる
            onChanged: (value) => ref.read(todosProvider.notifier).toggle(todo.id),
            title: Text(todo.description),
          ),
      ],
    );
  }
}
```




提供されたコードは、FlutterのReorderableTreeListViewのクラスと思われます。このコードに施せる主な最適化は以下の通りです。

ListViewの代わりにListView.builderを使用し、リストが長い場合にウィジェットの構築量を削減する。
リストが再構築されるたびにアイテムを再作成するのではなく、Keyを使用してリスト内のアイテムを識別する。
ステートフルウィジェットからノーティファイアに並び替えのロジックを移動し、ステートレスウィジェットがよりUIとレイアウトに集中できるようにする。
以下は、最適化されたバージョンの例です。

