


## CustomScrollViewとSliver

```dart
CustomScrollView(
      slivers: [
        SliverList(delegate: SliverChildListDelegate([
          //非SliverのWidgetを並べていく
          //静的に生成
        ])),
      ],
    );
```
```dart
SliverList(delegate: SliverChildBuilderDelegate((context, index) => {
  //リストに入れるWidgetを動的に生成
}))
```

## なぜSliverListとSliverGridを使用
>https://tech.excite.co.jp/entry/2022/01/11/000000
ListViewとGridViewはコンテンツを別々に表示するのには最適です。 しかし、もしリストやグリッドを一緒にスクロールさせたり、もっと複雑なスクロールをさせたい場合はSliverListとSliverGridを使用するのが最適です。
同時スクロールが可能に

## 数が多い場合にスムーズにスクロールさせたいとき、特に効果的
>https://bukiyo-papa.com/sliverlist-slivergrid/
Sliverはchildrenの数が多い場合にスムーズにスクロールさせたいとき、特に効果的です。

なぜなら、Sliverは遅延読み込みができ、それぞれの要素が表示されるそのときにビルドができるからです。

もし、最初の表示のときに見えていない部分も含めてすべての子供をビルドすると表示に時間がかかります。

見えている部分だけビルドする仕組みが「遅延読み込み」で、見えている部分しか表示の処理をしないので、画面がサクサク動きます。


SliverListはdelegateパラメータを取ります。
delegeteパラメータは、スクロールされて次のitemが表示されそうになったときにそのitemをビルドするための関数を設定します。

補足：delegateは直訳すると、「委譲」です。おまかせするという意味。
Viewを作るそのときに全てのアイテムをビルドするのではなく、アイテムが表示されそうになったタイミングでそのアイテムをビルドする。そのアイテムのビルドはdelegageパラメータに指定した関数に「おまかせ」します、という意味です。

Viewを作るそのときに全てのitemをビルドしてしまうとパフォーマンスがitem数に依存し、item数が多い場合にパフォーマンスが劣化してしまいます。

表示される分だけビルドするという仕組みにより、パフォーマンスが高く保たれ、それを実現するために重要なパラメータがdelegateパラメータです。

SliverChildListDelegateを使って実際のchildrenのリストを指定することもできますし、SliverChildBuilderDelegateを使ってitemをビルドする関数を指定することで遅延的に構築することもできます。
リッドの縦軸に関してはその他にも指定するパラメータがあります。

countコンストラクタを使ってそのグリッドがいくつのitemを持っているのかを指定します。

あるいは、グリッドにいくつのitemを表示するか決定するために、extentコンストラクタを使ってitemの最大幅を指定します


## ListView.separatedと同様の動作
```dart
import 'dart:math' as math;
import 'package:flutter/material.dart';

extension SliverListEx on SliverList {
  static SliverList separated({
    required int itemCount,
    required NullableIndexedWidgetBuilder itemBuilder,
    required NullableIndexedWidgetBuilder separatorBuilder,
  }) {
    return SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) {
          final itemIndex = index ~/ 2;
          return index.isEven 
	      ? itemBuilder(context, itemIndex)
	      : separatorBuilder(context, itemIndex);
        },
        childCount: math.max(0, itemCount * 2 - 1),
      ),
    );
  }
}
```

## ネストしたリスト - ShrinkWrapとSliverの比較
>https://dartpad.dev/workshops.html?webserver=https://fdr-shrinkwrap-slivers.web.app


### ネストされたリストを無造作に作る
ユーザーをジャンクな脅威から救う方法を見るには、ListViewsの代わりにSliverで同じUIを再構築するステップ2をチェックしてください。

## Sliversを使用したリストのリスト
右のコードは以前と同じUIを構築しますが、今回はシュリンクされたListViewオブジェクトの代わりにSliverを使用します。このページの残りの部分では、変更点をステップバイステップで説明します。

### ネストされたリストをSliversに移行する方法
ステップ1
まず、一番外側のListViewをSliverListに変更します。
```dart
// 前
@override
Widget build(BuildContext context) {
  return ListView.builder(
    itemCount: numberOfLists,
    itemBuilder: (context, index) => innerLists[index],
  );
}
```
になります。
```dart
// After
@override
Widget build(BuildContext context) {
  return CustomScrollView(slivers: innerLists);
}
```
### Step 2
次に、内側のリストのタイプを`List<ListView>`から`List<SliverList>`に変更します。
```dart
// Before
List<ListView> innerLists = [];
```
becomes:
```dart
// After
List<SliverList> innerLists = [];
```
### Step 3
さて、いよいよ内部のリストを再構築します。SliverListクラスは生のListViewクラスとは少し違っていて、主な違いはデリゲートの出現です。

変更後、ListViewオブジェクトはSliverListオブジェクトに置き換えられ、それぞれがSliverChildBuilderDelegateを使用して、効率的なオンデマンド構築を提供するようになりました。
#### After
```dart
@override
void initState() {
  super.initState();
  for (int i = 0; i < numLists; i++) {
    final _innerList = <ColorRow>[];
    for (int j = 0; j < numberOfItemsPerList; j++) {
      _innerList.add(const ColorRow());
    }
    innerLists.add(
      SliverList(
        delegate: SliverChildBuilderDelegate(
          (BuildContext context, int index) => _innerList[index],
          childCount: numberOfItemsPerList,
        ),
      ),
    );
  }
}
```
怠け者のビルディング、復活!
右のコードはすでにこの変更を適用しています。アプリを実行すると、Flutterが100個のColorRowウィジェットをすぐにレンダリングする必要がなくなったことに気づきます。スクロールすると、期待通りより多くのウィジェットが動的に構築されます。さらに良いことに、次のリストまでスクロールしても、特別なコストは発生しません。

Flutter は、必要なときに必要なだけウィジェットを作成するようになったのです。

```dart
class ShrinkWrapSlivers extends StatefulWidget {
  const ShrinkWrapSlivers({
    Key? key,
  }) : super(key: key);

  @override
  _ShrinkWrapSliversState createState() => _ShrinkWrapSliversState();
}

class _ShrinkWrapSliversState extends State<ShrinkWrapSlivers> {
  List<SliverList> innerLists = [];
  final numLists = 15;
  final numberOfItemsPerList = 100;

  @override
  void initState() {
    super.initState();
    for (int i = 0; i < numLists; i++) {
      final _innerList = <ColorRow>[];
      for (int j = 0; j < numberOfItemsPerList; j++) {
        _innerList.add(const ColorRow());
      }
      innerLists.add(
        SliverList(
          delegate: SliverChildBuilderDelegate(
            (BuildContext context, int index) => _innerList[index],
            childCount: numberOfItemsPerList,
          ),
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(slivers: innerLists);
  }
}

@immutable
class ColorRow extends StatefulWidget {
  const ColorRow({Key? key}) : super(key: key);

  @override
  State createState() => ColorRowState();
}

class ColorRowState extends State<ColorRow> {
  Color? color;

  @override
  void initState() {
    super.initState();
    color = randomColor();
  }

  @override
  Widget build(BuildContext context) {
    print('Building ColorRowState');
    return Container(
      decoration: BoxDecoration(
        gradient: LinearGradient(
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
          colors: [
            randomColor(),
            randomColor(),
          ],
        ),
      ),
      child: Row(
        children: <Widget>[
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Container(height: 50, width: 50, color: Colors.white),
          ),
          Flexible(
            child: Column(
              children: const <Widget>[
                Padding(
                  padding: EdgeInsets.all(8),
                  child: Text('I\'m a widget!',
                      style: TextStyle(color: Colors.white)),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

Color randomColor() =>
    Color((math.Random().nextDouble() * 0xFFFFFF).toInt()).withOpacity(1.0);

```









