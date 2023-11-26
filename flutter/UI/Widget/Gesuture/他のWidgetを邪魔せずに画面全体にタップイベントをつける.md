# 漫画アプリ(viewer)でスワイプ操作
>https://tech.excite.co.jp/entry/2022/10/25/145643

今回のユースケースとしては漫画アプリ(viewer)でスワイプ操作などをそのままに左側タップで次のページに進んだりするときの実装です。
通常Stackを用いてViewの上に GestureDetector を置いてしまうと全てのアクションが GestureDetector に吸われてしまいます。

## 実装
今回は画面全体として縦長に3つのGestureDetectorを置くという想定で説明していきます
```dart
Stack(
  children: [
    ComicView(),
    Row(
      children: [
        Flexible(
          child: GestureDetector(
            behavior: HitTestBehavior.translucent,
            onTap: () {} // 左側のタップイベント
            child: ConstrainedBox(
              constraints: const BoxConstraints.expand(),
            ),
          ),
        ),
        Flexible(
          child: GestureDetector(
            behavior: HitTestBehavior.translucent,
            onTap: () {} // 中央のタップイベント
            child: ConstrainedBox(
              constraints: const BoxConstraints.expand(),
            ),
          ),
        ),
        Flexible(
          child: GestureDetector(
            behavior: HitTestBehavior.translucent,
            onTap: () {} // 右側のタップイベント
            child: ConstrainedBox(
              constraints: const BoxConstraints.expand(),
            ),
          ),
        ),
      ],
    ),
  ],
),
```
まず GestureDetector にある behavior という引数に HitTestBehavior.translucent を渡してあげます。これによってComicViewの挙動を邪魔することは無くなります。
そしてchildに ConstrainedBox をいれ、 BoxConstraints.expand() を引数に入れます。何もWidgetがない場合に GestureDetector が表示する領域を確保できないからです。

## ConstrainedBoxについて
ConstrainedBox を使おうと思った経緯としては最初に SizedBox を使用してやろうと思っていたのですがうまくできませんでした。
Container を使用したところ上手くいったのですが Container では冗長的なので中身を見たところ最初に(厳密にはLimitedBoxを使用しているのですが) ConstrainedBox を使っていたので置き換えてみたところ上手くいきました。
名前の通りWidgetに対して制約を付けることができます。

































