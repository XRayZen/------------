# StreamProvider
- StreamProvider は FutureProvider の Stream 版です。

## 次のような用途で使われます
- Firebase や WebSocket の監視するため。
- 一定時間ごとに別のプロバイダを更新するため。

Stream 自体が値の更新を監視する性質を持つため、StreamProvider を使うことにあまり意義を見出せない方もいるかもしれません。 Flutter の StreamBuilder で十分ではないか、と。

## StreamProvider を使うことで得られる利点
- 他のプロバイダで`ref.watch`を通じてストリームを監視することができる。
- `AsyncValue` により `loading/error` のステートを適切に処理することができる。
- 通常のストリームとブロードキャスト（broadcast）ストリームを区別する必要がない。
- ストリームから出力された直近の値をキャッシュしてくれる（途中で監視を開始しても最新の値を取得することができる）。
- `StreamProvider`をオーバーライドすることでテスト時のストリームを簡単にモックすることができる。
## 使用例：ソケットを使用したライブチャット
StreamProviderは、ビデオストリーミングや天気予報など、非同期データのストリームを扱う場合に使用します。
```dart
final chatProvider = StreamProvider<List<String>>((ref) async* {
  // ソケットを使用してAPIに接続し、その出力をデコードする
  final socket = await Socket.connect('my-api', 4242);
  ref.onDispose(socket.close);
  
  var allMessages = const <String>[];
  await for (final message in socket.map(utf8.decode)) {
    // 新しいメッセージを受信しました。
    //全メッセージのリストに追加してみましょう。
    allMessages = [...allMessages, message];
    yield allMessages;
  }
});
```
そして、UIはこのようにライブストリーミングチャットを聴くことができます。
```dart
Widget build(BuildContext context, WidgetRef ref) {
  final liveChats = ref.watch(chatProvider);
  // FutureProviderと同様に、AsyncValue.whenを使用して
  //ロード/エラー状態を処理することが可能です。
  return liveChats.when(
    loading: () => const CircularProgressIndicator(),
    error: (error, stackTrace) => Text(error.toString()),
    data: (messages) {
      // すべてのメッセージをスクロール可能なリストビューで表示します。
      return ListView.builder(
        // メッセージを下から上へ表示する
        reverse: true,
        itemCount: messages.length,
        itemBuilder: (context, index) {
          final message = messages[index];
          return Text(message);
        },
      );
    },
  );
}
```



