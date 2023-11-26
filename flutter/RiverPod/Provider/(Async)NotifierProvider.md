# (非同期)NotifierProvider
- Notifier/AsyncNotifier を監視して公開するために使用されるプロバイダです。
- AsyncNotifier は、非同期に初期化することができる Notifier です。
- (Async)NotifierProviderと(Async)Notifierは、ユーザーの操作に反応して変化する状態を管理するためのRiverpodの推奨ソリューションである。

## 次のような用途に使用
- カスタムイベントに反応した後、時間の経過とともに変化する可能性のある状態を公開する。
- 状態を変更するためのロジック（別名「ビジネスロジック」）を一箇所に集中させ、長期的な保守性を向上させる。

## コード生成を使わない
>https://zenn.dev/flutteruniv_dev/articles/a8f10da7e80ed0
を参考にカスタム
```dart
// AsyncNotifierを使うためのAsyncNotifierProviderを定義する.
final searchRequestProvider =
    AsyncNotifierProvider<SearchNotifier, SearchRequest>(() {
  return SearchNotifier();
});

class SearchNotifier extends AsyncNotifier<SearchRequest> {
  @override
  FutureOr<SearchRequest> build() {
    return SearchRequest(searchType: SearchType.addContent, word: 'word');
  }

  Future<void> add(SearchRequest re) async {
    //Stateの状態を読み込みモードにする
    state = const AsyncValue.loading();
    // stateを更新
    state = await AsyncValue.guard(() async {
      return re;
    });
  }
}
```
!!! tip 利用方法
    同期コード内でも非同期関数を呼んでも良い
    ```dart
    ref.watch(searchRequestProvider.notifier).add(
      SearchRequest(searchType: SearchType.addContent,word: txt,),);
    ```
## 使用例 AsyncNotifierProvider
- リモートTodoリストを実装することができます。
- そうすることで、addTodoのようなメソッドを公開し、ユーザーとのインタラクションでUIがTodoリストを変更できるようにすることができます。
```dart
@freezed
class Todo with _$Todo {
  factory Todo({
    required String id,
    required String description,
    required bool completed,
  }) = _Todo;

  factory Todo.fromJson(Map<String, dynamic> json) => _$TodoFromJson(json);
}

// これにより、AsyncNotifierとAsyncNotifierProviderが生成されます。
// AsyncNotifierProviderに渡されるAsyncNotifierクラスです。
// このクラスは、"state" プロパティ以外の状態を公開してはいけません!
// このクラスのパブリックメソッドは、UIが状態を変更できるようにするものです。
// 最後に、asyncTodosProvider(AsyncNotifierProvider) を使用して、
//UI が Todos クラスと対話できるようにしています。
@riverpod
class AsyncTodos extends _$AsyncTodos {
  Future<List<Todo>> _fetchTodo() async {
    final json = await http.get('api/todos');
    final todos = jsonDecode(json) as List<Map<String, dynamic>>;
    return todos.map((todo) => Todo.fromJson(todo)).toList();
  }

  @override
  FutureOr<List<Todo>> build() async {
    //リモートリポジトリから初期Todoリストを読み込む
    return _fetchTodo();
  }

  Future<void> addTodo(Todo todo) async {
    // 状態をロードに設定する
    state = const AsyncValue.loading();
    // 新しいTodoを追加し、リモートリポジトリからTodoリストを再読み込みします。
    state = await AsyncValue.guard(() async {
      await http.post('api/todos', todo.toJson());
      return _fetchTodo();
    });
  }

  // Let's allow removing todos
  Future<void> removeTodo(String todoId) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await http.delete('api/todos/$todoId');
      return _fetchTodo();
    });
  }

  // Let's mark a todo as completed
  Future<void> toggle(String todoId) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await http.patch(
        'api/todos/$todoId',
        <String, dynamic>{'completed': true},
      );
      return _fetchTodo();
    });
  }
}
```
- AsyncNotifierProviderを定義したので、それを使ってUIでTODOのリストと対話することができます。
```dart
class TodoListView extends ConsumerWidget {
  const TodoListView({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // rebuild the widget when the todo list changes
    final asyncTodos = ref.watch(asyncTodosProvider);

    // Let's render the todos in a scrollable list view
    return asyncTodos.when(
      data: (todos) => ListView(
        children: [
          for (final todo in todos)
            CheckboxListTile(
              value: todo.completed,
              // TODOをタップすると、その完了状態を変更する
              onChanged: (value) =>
                  ref.read(asyncTodosProvider.notifier).toggle(todo.id),
              title: Text(todo.description),
            ),
        ],
      ),
      loading: () => const Center(
        child: CircularProgressIndicator(),
      ),
      error: (err, stack) => Text('Error: $err'),
    );
  }
}
```

## 使用例 (同期的)NotifierProvider
- Todo-Listを実装するためにNotifierProviderを使用することができます。
- そうすることで、addTodoのようなメソッドを公開し、ユーザーの操作に応じてUIがTodoのリストを変更できるようにします。
```dart
@freezed
class Todo with _$Todo {
  factory Todo({
    required String id,
    required String description,
    required bool completed,
  }) = _Todo;
}

// これにより、NotifierとNotifierProviderが生成される。
// NotifierProviderに渡されるNotifierクラス
// このクラスは、"state" プロパティ以外の状態を公開してはいけない
//このクラスのパブリックメソッドは、UIが状態を変更できるようにするもの
// 最後に、todosProvider(NotifierProvider) を使って、
//UI が Todos クラスと対話できるようにしています。
@riverpod
class Todos extends _$Todos {
  @override
  List<Todo> build() {
    return [];
  }

  // UIでTODOを追加できるようにしよう。
  void addTodo(Todo todo) {
    // 状態は不変なので、`state.add(todo)`は許されない。
    // 代わりに、前の項目と新しい項目を含む新しいTODOリストを作成する必要があります。
    // ここでDartのスプレッド演算子を使うと便利です!
    state = [...state, todo];
    // notifyListeners」などを呼び出す必要はない。state =」を呼べば、
    //必要なときに自動的にUIを再構築してくれる。
  }

  // Let's allow removing todos
  void removeTodo(String todoId) {
    // ここでも、状態は不変である。つまり、既存のリストを変更するのではなく、新しいリストを作っているのです。
    state = [
      for (final todo in state)
        if (todo.id != todoId) todo,
    ];
  }

  // ToDoを完了マークしてみよう
  void toggle(String todoId) {
    state = [
      for (final todo in state)
        // 一致するTODOのみを完了としてマークしています。
        if (todo.id == todoId)
          //もう一度、状態は不変なので、Todoのコピーを作成する必要があります。
          //そのために、以前実装した `copyWith` メソッドを使用します。
          todo.copyWith(completed: !todo.completed)
        else
          //他のTODOは変更されない
          todo,
    ];
  }
}
```
- NotifierProviderを定義したので、それを使ってUIでTODOのリストと対話することができます。
```dart
class TodoListView extends ConsumerWidget {
  const TodoListView({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ToDoリストが変更されたときにウィジェットを再構築する
    List<Todo> todos = ref.watch(todosProvider);

    // スクロール可能なリストビューでTODOをレンダリングしよう
    return ListView(
      children: [
        for (final todo in todos)
          CheckboxListTile(
            value: todo.completed,
            // TODOをタップすると、その完了状態を変更する
            onChanged: (value) =>
                ref.read(todosProvider.notifier).toggle(todo.id),
            title: Text(todo.description),
          ),
      ],
    );
  }
}
```









