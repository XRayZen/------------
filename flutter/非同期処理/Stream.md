# 非同期データストリーム(Stream)

><https://www.youtube.com/watch?v=nQBpOIHE4eE&t=276s>
- Futureとは違い、Streamは値もしくはエラーを経時的に返してくる
- 非同期な往復子(iterator)を返してくるイメージ
- listen() メソッドを使うことで、新しい値が追加されたタイミングでその値を使って何かしらの処理を行うリスナー(メソッド)を設定できる
- 連続した値をメソッドにより変換（例えば、連続する全ての値を２倍にしたり、連続で同じ値が来た場合はそれを取り除いたり）できる。
- ユーザからのアクションに応じた処理、web APIを用いる際などの非同期処理をうまく扱うことができる
- メソッド（オペレータ）でデータの変換処理などができ、メソッドチェーンを使った読みやすいコードを書くことができる
- デフォルトのストリームはリスナーはライフを通じて一つだけ
- 複数の場合はブロードキャストストリーム

>ストリームの作成例
```dart
Stream<int> count() async*{
    int i=0;
    While(true){
        yield i++;
    }
}
```
## listenメソッド
>ストリームの受信
- listenメソッドで受信した値を処理するメソッドを設定
```dart
final subsc= mystream.listen((value) {
    #受信した値の処理
  }, onError: (error) {
    #エラー処理
  },
  cancelOnError: flase,//これだとエラー出ても継続
   onDone: () {
    # ストリームが終わったときの処理
    print('Streamをcloseしました');
  });
```
## 同期的に受け取る
```dart
await stream.forEach((element) => print(element));
await for(int i in stream){
  print(i);
}
```
### リッスンメソッドのリターンの使い方
- ストリームの停止・開始・中止が出来る
```dart
subsc.pause();
subsc.resume();
subsc.cancel();
```

## streamはiterなのでmapなどメソッドチェーンも使える
- 詳細は割愛
```dart
mystream.map((x)=>x.index==0)
.listen(print);
```

### StreamBuilder

- 登録されたstreamの値が更新されるたびにbuilder以下を再生成してくれる。

### BroadcastStream

- BroadcastStreamは、複数回listen()することができるStream。
- broadcastでないStream（デフォルト）は、複数回listen()する場合、２回目以降は何も起きない
