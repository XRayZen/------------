# Flutter Hooks #
A Flutter implementation of React hooks: https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889

フックは、ウィジェットのライフサイクルを管理する新しい種類のオブジェクトです。その理由は、重複を排除することでWidget間のコード共有を促進するためです。

## 動機
StatefulWidget には大きな問題があります。それは initState や dispose などのロジックを再利用することが非常に難しいということです。例えば、AnimationController がその例だ。
```dart
class Example extends StatefulWidget {
  final Duration duration;

  const Example({Key? key, required this.duration})
      : super(key: key);

  @override
  _ExampleState createState() => _ExampleState();
}

class _ExampleState extends State<Example> with SingleTickerProviderStateMixin {
  AnimationController? _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this, duration: widget.duration);
  }

  @override
  void didUpdateWidget(Example oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.duration != oldWidget.duration) {
      _controller!.duration = widget.duration;
    }
  }

  @override
  void dispose() {
    _controller!.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```
AnimationController を使用したいすべてのウィジェットは、このロジックのほとんどを一から再実装する必要がある。

Dart mixin はこの問題を部分的に解決することができますが、他の問題にも悩まされます。

あるミキシンはひとつのクラスにつき一度しか使用できない。
ミキシンとクラスは同じオブジェクトを共有します。
つまり、2つのmixinが同じ名前で変数を定義した場合、コンパイルに失敗して動作が不明なまま結果が異なる可能性があります。
本ライブラリは、第三の解決策を提案する。
```dart
class Example extends HookWidget {
  const Example({Key? key, required this.duration})
      : super(key: key);

  final Duration duration;

  @override
  Widget build(BuildContext context) {
    final controller = useAnimationController(duration: duration);
    return Container();
  }
}
```
このコードは、機能的には前の例と同じです。AnimationController を破棄し、Example.duration が変更されたときに duration を更新していることに変わりはありません。しかし、おそらくあなたはこう思うでしょう。

このロジックはどこに行ったのだろう？

このロジックは useAnimationController に移動しました。このライブラリに直接含まれる関数です（既存のフックを参照）。

フックは新しい種類のオブジェクトで、いくつかの特殊性があります。

Hook は新しい種類のオブジェクトで、いくつかの特殊性があります：Hook をミックスしたウィジェットのビルドメソッドでのみ使用できます。

同じフックを何度でも再利用することができる。次のコードでは、2 つの独立した AnimationController を定義しています。これらは、ウィジェットの再構築時に正しく保持されます。
```dart
Widget build(BuildContext context) {
  final controller = useAnimationController();
  final controller2 = useAnimationController();
  return Container();
}
```
フックは，互いに，またウィジェットから完全に独立しています．
つまり、簡単にパッケージに抽出し、他の人が使えるようにパブで公開することができます。

### 原則
Stateと同様に、フックはWidgetのElementに格納されます。しかし、1つのStateを持つ代わりに、ElementはList<Hook>を格納する。そして、Hookを使用するためには、Hook.useを呼び出す必要があります。

useによって返されるHookは、それが呼ばれた回数に基づきます。最初の呼び出しは最初のフックを返し、2回目の呼び出しは2番目のフックを返し、3回目の呼び出しは3番目のフックを返し、といった具合になる。

この考え方がまだはっきりしない場合、フックの素朴な実装は次のようになる だろう。
```dart
class HookElement extends Element {
  List<HookState> _hooks;
  int _hookIndex;

  T use<T>(Hook<T> hook) => _hooks[_hookIndex++].build(this);

  @override
  performRebuild() {
    _hookIndex = 0;
    super.performRebuild();
  }
}
```
For more explanation of how hooks are implemented, here's a great article about how is was done in React: https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e

Rules 
Due to hooks being obtained from their index, some rules must be respected:

DO always prefix your hooks with use: 
```dart
Widget build(BuildContext context) {
  // starts with `use`, good name
  useMyHook();
  // doesn't start with `use`, could confuse people into thinking that this isn't a hook
  myHook();
  // ....
}
```
DO call hooks unconditionally 
```dart
Widget build(BuildContext context) {
  useMyHook();
  // ....
}
```
DON'T wrap use into a condition 
```dart
Widget build(BuildContext context) {
  if (condition) {
    useMyHook();
  }
  // ....
}
```
## About hot-reload 
Since hooks are obtained from their index, one may think that hot-reloads while refactoring will break the application.

But worry not, a HookWidget overrides the default hot-reload behavior to work with hooks. Still, there are some situations in which the state of a Hook may be reset.

Consider the following list of hooks:

useA();
useB(0);
useC();
Then consider that we edited the parameter of HookB after performing a hot-reload:

useA();
useB(42);
useC();
Here everything works fine and all hooks maintain their state.

Now consider that we removed HookB. We now have:

useA();
useC();
In this situation, HookA maintains its state but HookC gets hard reset. This happens because, when a hot-reload is performed after refactoring, all hooks after the first line impacted are disposed of. So, since HookC was placed after HookB, it will be disposed.

How to create a hook 
There are two ways to create a hook:

A function

Functions are by far the most common way to write hooks. Thanks to hooks being composable by nature, a function will be able to combine other hooks to create a more complex custom hook. By convention, these functions will be prefixed by use.

The following code defines a custom hook that creates a variable and logs its value to the console whenever the value changes:
```dart
ValueNotifier<T> useLoggedState<T>([T initialData]) {
  final result = useState<T>(initialData);
  useValueChanged(result.value, (_, __) {
    print(result.value);
  });
  return result;
}
```
A class

When a hook becomes too complex, it is possible to convert it into a class that extends Hook - which can then be used using Hook.use.
As a class, the hook will look very similar to a State class and have access to widget life-cycle and methods such as initHook, dispose and setState.

It is usually good practice to hide the class under a function as such:
```dart
Result useMyHook() {
  return use(const _TimeAlive());
}
```
The following code defines a hook that prints the total time a State has been alive on its dispose.
```dart
class _TimeAlive extends Hook<void> {
  const _TimeAlive();

  @override
  _TimeAliveState createState() => _TimeAliveState();
}

class _TimeAliveState extends HookState<void, _TimeAlive> {
  DateTime start;

  @override
  void initHook() {
    super.initHook();
    start = DateTime.now();
  }

  @override
  void build(BuildContext context) {}

  @override
  void dispose() {
    print(DateTime.now().difference(start));
    super.dispose();
  }
}
```
## Existing hooks 
Flutter_Hooks already comes with a list of reusable hooks which are divided into different kinds:

Primitives 
A set of low-level hooks that interact with the different life-cycles of a widget

Name	Description
useEffect	Useful for side-effects and optionally canceling them.
useState	Creates a variable and subscribes to it.
useMemoized	Caches the instance of a complex object.
useRef	Creates an object that contains a single mutable property.
useCallback	Caches a function instance.
useContext	Obtains the BuildContext of the building HookWidget.
useValueChanged	Watches a value and triggers a callback whenever its value changed.
Object-binding 
This category of hooks the manipulation of existing Flutter/Dart objects with hooks. They will take care of creating/updating/disposing an object.

dart:async related hooks:
Name	Description
useStream	Subscribes to a Stream and returns its current state as an AsyncSnapshot.
useStreamController	Creates a StreamController which will automatically be disposed.
useFuture	Subscribes to a Future and returns its current state as an AsyncSnapshot.
Animation related hooks:
Name	Description
useSingleTickerProvider	Creates a single usage TickerProvider.
useAnimationController	Creates an AnimationController which will be automatically disposed.
useAnimation	Subscribes to an Animation and returns its value.
Listenable related hooks:
Name	Description
useListenable	Subscribes to a Listenable and marks the widget as needing build whenever the listener is called.
useListenableSelector	Similar to useListenable, but allows filtering UI rebuilds
useValueNotifier	Creates a ValueNotifier which will be automatically disposed.
useValueListenable	Subscribes to a ValueListenable and return its value.
Misc hooks:
A series of hooks with no particular theme.

Name	Description
useReducer	An alternative to useState for more complex states.
usePrevious	Returns the previous argument called to [usePrevious].
useTextEditingController	Creates a TextEditingController.
useFocusNode	Creates a FocusNode.
useTabController	Creates and disposes a TabController.
useScrollController	Creates and disposes a ScrollController.
usePageController	Creates and disposes a PageController.
useAppLifecycleState	Returns the current AppLifecycleState and rebuilds the widget on change.
useOnAppLifecycleStateChange	Listens to AppLifecycleState changes and triggers a callback on change.
useTransformationController	Creates and disposes a TransformationController.
useIsMounted	An equivalent to State.mounted for hooks.
useAutomaticKeepAlive	An equivalent to the AutomaticKeepAlive widget for hooks.
useOnPlatformBrightnessChange	Listens to platform Brightness changes and triggers a callback on change.
Contributions 
Contributions are welcomed!

If you feel that a hook is missing, feel free to open a pull-request.

For a custom-hook to be merged, you will need to do the following:

Describe the use-case.

Open an issue explaining why we need this hook, how to use it, ... This is important as a hook will not get merged if the hook doesn't appeal to a large number of people.

If your hook is rejected, don't worry! A rejection doesn't mean that it won't be merged later in the future if more people show interest in it. In the mean-time, feel free to publish your hook as a package on https://pub.dev.

Write tests for your hook

A hook will not be merged unless fully tested to avoid inadvertently breaking it in the future.

Add it to the README and write documentation for it.

