
!!! tip Widgetの単体テストはマテリアルappで囲む必要がある
# 関数解説
## 描画するまで待つ
```dart
await tester.pumpAndSettle();
```

# Flutter ウィジェットテスト入門
>https://future-architect.github.io/articles/20210519a/
>
デフォルトで作成されたWidgetテストファイルを解説
## testWidgets
先頭で使用されているのが testWidgets() という関数です。
```dart {.line-numbers}
void main() {
  testWidgets('Counter increments smoke test', (WidgetTester tester) async {
    // ...
  }
  // ...
}
```
これは flutter_test パッケージで定義されている関数で、ウィジェットテストを実施したい時に使うものです。 WidgetTester というヘルパークラスが用意されており、実際のテストコードはこの WidgetTester を活用しつつ記述します。
## pumpWidget
次は pumpWidget() についてです。 testWidgets() の先頭に登場する関数です。
```dart {.line-numbers}
void main() {
  testWidgets('Counter increments smoke test', (WidgetTester tester) async {
    // Build our app and trigger a frame.
    await tester.pumpWidget(MyApp());
```
pumpWidget() は対象の Widget のインスタンスを生成し、その生成処理が問題なく完了することをチェックします。今回は MyApp が指定されているので、 main.dart にて StatelessWidget として定義されている MyApp がチェック対象です。
## pump
pump() は Widget の再生成を促すメソッドです。通常UI操作により描画対象に変更が加わった際には自動的に Widget が再生成されます。
![](https://future-architect.github.io/images/20210519a/image_2.png)
例えば以下の画面。右下のボタンを2回押下したのですが、画面中央の数字がボタン押下に合わせて増えています。ユーザの操作に合わせて Widget の再生成が行われています。
## find
- UIコンポーネントを含むテストで一番大変なのは、テスト対象のオブジェクトをテストプログラム上で特定すること
- これが簡単にできるように整備されているほどテスタビリティが高い

find() は flutter_test パッケージが提供するトップレベルの関数で、文字やアイコンなどを元に該当する Widget を特定する Finder として機能します。

今回参照しているテストの中でも数箇所で登場します。
```dart
// // Verify that our counter starts at 0.
expect(find.text('0'), findsOneWidget);
expect(find.text('1'), findsNothing);

// // Tap the '+' icon and trigger a frame.
await tester.tap(find.byIcon(Icons.add));
```
find.text('0') というのが Finder で、0 の文字列を含む Widget を Widget ツリーの中から探索します。今回は画面中央に表示されるカウンターがヒットします。

また、 find.byIcon(Icons.add) ではアイコンを起点に Widget を探索します。今回は画面右下に表示されるプラスマークの書かれた青いボタンがヒットします。

## expect
expects() は Matcher と一緒に用いることで、Widget が期待通りに生成されているか否かを検証します。

今回だとまずは以下の部分。初期描画時は画面中央のカウンターは 0 と表記されているはずです。
```dart
// // Verify that our counter starts at 0.
expect(find.text('0'), findsOneWidget);
expect(find.text('1'), findsNothing);
```
次に以下の部分。ボタンを1回押下するので、カウンターの数値は 0 から 1 に変わっているはずです。ちなみに、先程記載したようにボタン押下後には Widget 再生成が行われないので pump() をコールしています。
```dart
// // Tap the '+' icon and trigger a frame.
await tester.tap(find.byIcon(Icons.add));
await tester.pump();

// Verify that our counter has incremented.
expect(find.text('0'), findsNothing);
expect(find.text('1'), findsOneWidget);
```
これでテストファイルの中身はすべて触れることができました。

## entertext
finder]で指定されたテキスト入力ウィジェットにフォーカスを与え、その内容をあたかもオンスクリーンキーボードから入力されたように[text]で置き換えます。

finder]で指定されたウィジェットは、[EditableText]または[EditableText]の子孫である必要があります。例えば find.byType(TextField) や find.byType(TextFormField) 、あるいは find.byType(EditableText) などです。

返された未来が完了すると、テキスト入力ウィジェットのテキストは正確にテキストになり、キャレットはテキストの末尾に配置されます。

テキストを入力せずに [finder] にだけフォーカスを当てるには、[showKeyboard] を参照してください。

他のウィジェット (例えば、[EditableText] と同じように TextInputConnection を維持するカスタム ウィジェット) にテキストを入力するには、まず、そのウィジェットがオープン接続であることを確認してから (例えば、フォーカスを当てるために [tap] を使用して)、 testTextInput.enterText を直接呼び出します ([TestTextInput.enterText] を参照してください)。


## stub
### thenReturn
このメソッドスタブに対する定型応答を格納する。

注: ゾーンの関係で、[expected] は Future や Stream にはできません。メソッドスタブからFutureまたはStreamを返すには、[thenAnswer]を使用します。

## アノテーション
### @GenerateNiceMocks
///Mockitoにモッククラスの生成を指示するためのアノテーションです。

 コード生成][NULL_SAFETY_README]の際に、Mockitoはモック対象となる各クラスに対して `Mock{Type} extends Mock` クラスを `{name}.mocks.dart` (`{name}` は `@GenerateNiceMocks` が使われているファイルのベースネーム)に生成します。
///
 例えば、`@GenerateNiceMocks([MockSpec<Foo>()])` がDartライブラリのトップレベルである `foo_test.dart` にあった場合、Mockitoは `クラスMockFoo extends Mock implements Foo` を新しいライブラリ `foo_test.mocks.dart` の中に生成することになります。
///
/// もし、モックとなるクラスがジェネリックであれば、モックも同じようにジェネリックになります。
/// 例えば、クラス `class Foo<T, U>` が与えられた場合、Mockito は `class MockFoo<T, U> extends Mock implements Foo<T, U>` を生成します。
///
/// `@GenerateNiceMocks` は `@GenerateMocks` と2つの点で異なります。
/// - 引数リストには `MockSpec` のみを指定することができます。
/// - 生成されたモックは、デフォルトではスタブしていないメソッドの呼び出しではスローされず、代わりにターゲットの型に適した値が返されます。
///
/// [null_safety_readme]: https://github.com/dart-lang/mockito/blob/master/NULL_SAFETY_README.md


## テスター関数
```dart
await tester.sendKeyDownEvent(LogicalKeyboardKey.enter);
await tester.testTextInput.receiveAction(TextInputAction.done);
```

