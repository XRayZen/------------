# FlutterでIsolateを用いた並列処理をするべきシーンとそのやり方
# 結論
- 必要に応じて compute 関数で並列処理をすると、基本シングルスレッドのFlutterでもユーザー体験を損ねることなくヘビーな処理をさばける
## 概要と説明
- 操作感の良いアプリとするためには、ユーザー操作に即座に反応し、滑らかな画面更新がなされることが重要
- モバイル端末の多くは60fps(1秒間に60回の頻度)で画面更新をするようになっていて(iPad Proなど120fpsの端末も一部存在
- つまりアプリケーションコードはそれに間に合うように約16ms以内に処理を実行する必要がある
- これを超えると、レスポンスがワンテンポ遅れてもっさり感がでたり、画面がカクついたぎこちないものとなって、使用感が下がる
## TL;DR
- 大前提として、Dartの実行モデルは**シングルスレッド・イベントループ**である
- Main Isolateだけでは Futures・Streams APIなどを用いて並行処理はできるが並列処理はできない
- ネットワーク通信のレスポンス待ちなどのCPU負荷が低く単に長い待ち時間が発生するだけの処理は、単にMain Isolate上での並行処理で済ませるだけで良い
- しかし、16ms(60fps端末の場合)を超える処理時間を要するようなCPU負荷の特に高い処理をUIのフレームレートを落とさずに行うには、別のIsolateを起動して並列処理する必要がある
- Isolate を直接扱うとコードが煩雑になるので、それをラップした compute 関数で済む場合はそれを使うのが良い
- ただし、別Isolateの利用にはスイッチングコスト(10–20ms以上程度)がかかるので、それは意識した方が良い(巷の軽量なスレッドよりはコストが高い)
## Dartの実行モデルはシングルスレッド・イベントループである
>The Event Loop and Dart (翻訳) - Qiita  
>https://qiita.com/takyam/items/6ad155678c95bba4047f
- シングルスレッド・イベントループという実行モデルは、Node.jsと同様なので、豊富にあるこういうNode.jsについての説明を併読すると理解しやすい
>https://blog.hiroppy.me/entry/nodejs-event-loop

>イベントループとは、 JavaScriptがシングルスレッドなのにもかかわらず、効率よくノン ブロッキングI/Oを実行できるようにする仕組み  
イベントループはメインスレッドで実行される

- 要点としては、次の図のイメージが掴めれば十分
  
![The Event Loop and Dart](images/The%20Event%20Loop%20and%20Dart%20より.webp)[flutter Event loop]
- Main isolate: 通常の処理はすべてこのシングルスレッド上で実行される
- キューに溜まったイベントが順に実行される

- ゆえに、もしあるイベントの処理時間が長いと、その間は後続の溜まっているイベントが一切処理されない
-  シングルスレッドなので、UI更新など含めてあらゆる処理が詰まってしまう

というわけで、それらをどう対処すれば良いのかを説明していく
### CPU負荷が低く単に長い待ち時間が発生するだけの処理はFutureで捌くだけで良い
- Flutterアプリにおける大半の処理はこちらに分類される  
典型的な例としては、ネットワーク通信のレスポンス待ちなどが該当

例えば、 http パッケージの get関数 は次のようにFutureに包んだResponseを返す
```dart
Future<Response> get(url, {Map<String, String> headers}) =>
    _withClient((client) => client.get(url, headers: headers));
```
FutureはJavaScriptのPromiseとほぼ同様のものと捉えてOK   
利用する際は、次のようにコールバック形式でも良いが、
```dart
http.get("http://example.com/xxx").then((response) {
  // レスポンスが返ってきた後に時間差で呼ばれる
});
 // 同期的にすぐ実行される
```
async/await を活用して次のようにあたかも同期処理かのように書けるため、基本的にはこう書いた方がコードがすっきりして良い
```dart
var response = await http.get("http://example.com/xxx");
// レスポンスが返ってきた後に時間差で呼ばれる
```
## Futureの処理の流れ
- ただ、処理の流れをイメージする場合は、前者のthenを用いたコールバック形式の方が分かりやすいはず  
- ざっくり次のような流れで、Main Isolateのシングルスレッドを詰まらせることなく時間のかかる処理が捌かれる
  1. http.get の戻り値の Future<Response> に対して、 .then でレスポンスが返ってきた後の処理を登録する
  2. レスポンスはイベントループ中に定期的にポーリングされ、実際に返ってきたタイミングで .then に登録した処理が継続される
- Dartの実行モデルはシングルスレッド・イベントループのため、即座に結果を得られない可能性がある関数の戻り値は大抵 Future 型となっている
- つまり、普通に素直なコードを書いていれば、スレッドをブロックしてしまうようなコードを書く方が難しい作りになっている 

あるいは、ioパッケージの下記メソッドのように非同期版と同期版が用意されている場合もあるが、名前的に「デフォルトは非同期で同期版も用意」という形になっていることからもなるべく非同期版を使ってスレッドをブロックさせないような設計になっている
   - read メソッド: 非同期版(結果をFuture型でラップ)
   - readSync メソッド: 同期版  

同期版はCLIツールなど後続の処理がブロックされても問題ない場合に使うのはありだが、async/awaitがサポートされて非同期処理も同期的な書き方ができるようになった今となっては同期版を別途用意する意義はほとんど無い
## CPU負荷が高く時間のかかる連続的な処理はスレッドがブロックされる問題
上の例では、HTTPリクエストのレスポンス待ちという、CPUはほとんど仕事をせずに結果が来るまでほぼ待っているだけで良い類の処理だったが、一方CPUが占有されるような処理の場合はその間どうしてもスレッドがブロックされることになる

- Flutterアプリでよくありそうな例としては、APIレスポンスのパース
- 例として、次のようなユーザー情報100要素から成るJSONレスポンス(サイズは約15KB)のパース
```json
{"id":"acc4ab51-bf29-90bc-315e-c7ee959e61fe","name":"Edythe Schultz IV","email":"windler_torrey@wuckert.info","createdAt":"2019-02-02T14:01:31.860318"}
```
>実際のJSONレスポンス: http://www.mocky.io/v2/5c55243c2f00005000bf758a
- 次のようなUserクラスを用意
```dart
class User {
  User({
    @required this.id,
    @required this.name,
    @required this.email,
    @required this.createdAt,
  });
  final String id;
  final String name;
  final String email;
  final DateTime createdAt;

  User.fromJson(Map<String, dynamic> json)
      : id = json['id'] as String,
        name = json['name'] as String,
        email = json['email'] as String,
        createdAt = DateTime.parse(json["createdAt"] as String);
}
```
次のようにレスポンスをパースしてみます
```dart
Future<List<User>> _fetchUsers() async {
  final response =
      await http.get('http://www.mocky.io/v2/5c55243c2f00005000bf758a');
  final stopwatch = Stopwatch()..start();
  final users = _parse(response.body);
  stopwatch.stop();
  print('elapsed: ${stopwatch.elapsedMilliseconds}');
  return users;
}

List<User> _parse(String body) {
  final json = (jsonDecode(body) as List).cast<Map<String, dynamic>>();
  return json.map((j) => User.fromJson(j)).toList();
}
```
- 上記のように便利なStopwatchクラスを用いてiPhone 7 Plus実機でリリースモードで計測したところ、15ms前後
- 60fpsを満たすための16msギリギリでけっこう危うい
- ただ、実機での操作を試したところ、このように「1フレーム欠落するかも」程度の処理では体感として全く違和感ないレベル
- ただ、性能の低い端末では懸念すべきであり、また実際のアプリではもっとレスポンスサイズが増える
- よって、レスポンスパース処理でのメインスレッドブロックは考慮すべきポイントの1つである

ここではレスポンスのパースを一例としてあげたが、他にもヘビーな計算などアプリ要件によって色々あり得る

- というわけで、シングルスレッドのイベントループモデルはこういった計算負荷が重いケースに対して相性が良くない
- ただ、成す術がないわけではなくDartの場合はMain 以外のIsolateで処理する方法がある
## 非Main Isolateを用いた並列処理
- Main Isolate上でのFutureを用いた処理はシングルスレッド上での並行処理でしたが、
- Main以外のIsolateを起動してそこで処理を行うことでDartでもマルチスレッドを用いた並列処理ができます。
- 並行処理・並列処理の違いについては、 下記URLなど参照
- 次の図を見るだけでも分かりやすい
> 並行処理、並列処理のあれこれ:
>https://qiita.com/Kohei909Otsuka/items/26be74de803d195b37bd
![event](images/並行処理、並列処理のあれこれ%20より.webp)

- Isolateは次のような特徴を持っていて、いわゆるスレッドなどとは異なります。
  - それぞれがイベントループを持つ
  - Isolate同士でメモリーを共有することはできず、データはメッセージによって交換する
- Isolateの実体はプラットフォームごとに異なります。
  - iOSでは新しいIsolateを起動する度にスレッドが起動したり、たまに既存のスレッドが再利用されることが観測されました。
## 実際に非MainのIsolate上でパース処理をしてみる
非MainのIsolate上でのパース処理は、次のように書けます。
```dart
import 'dart:isolate';

Future<List<User>> _fetchUsersIsolate() async {
  final response =
      await http.get('http://www.mocky.io/v2/5c55243c2f00005000bf758a');
  final receivePort = ReceivePort();
  await Isolate.spawn(_isolate, receivePort.sendPort);
  final sendPort = await receivePort.first as SendPort;
  final answer = ReceivePort();
  sendPort.send([response.body, answer.sendPort]);
  final users = await answer.first as List<User>;
  return users;
}

void _isolate(SendPort initialReplyTo) {
  final receivePort = ReceivePort();
  initialReplyTo.send(receivePort.sendPort);
  receivePort.listen((message) {
    final data = message[0] as String;
    final send = message[1] as SendPort;
    send.send(_parse(data));
  });
}
```
次のような処理の流れになっています。
1. Isolate.spawnにて新しいIsolateを起動
2. 起動されたIsolateではデータ(この場合JSONレスポンス)が送られてきたらパース処理をするように登録
3. Main Isolateはポートを通じて、JSONレスポンスを送って、パース結果を非同期で受け取る  

プリミティブな処理で煩雑で、型の取り違えによる実行時エラーなども容易に発生しそうなコードです🤔

## compute関数を活用
FlutterではIsolateを手軽に扱えるようにcompute関数が用意されているので、通常はIsolateを直接使うのではなくそれを活用します。

すると、Isolateを扱う箇所はたった1行で書けるようになりました🎉
```dart
Future<List<User>> _fetchUsersCompute() async {
  final response =
      await http.get('http://www.mocky.io/v2/5c55243c2f00005000bf758a');
  final users = await compute(_parse, response.body);
  return users;
}
```
compute 関数は次のように定義されていて、タイプセーフに扱える
```dart
Future<R> compute <Q, R>(
  ComputeCallback<Q, R> callback,
  Q message, {
  String debugLabel
})
```
- コードの見た目としては、巷のスレッドを扱うメソッドくらいの感覚で使えるようになった感がありますが、多少癖や制約があります。
- 簡単に扱えるといってもIsolateのラッパーなので、上でIsolate APIを直接使って書いた例と照らし合わせると納得できるはずです。
- compute 関数の第一引数に処理したい関数、第二引数に渡したい引数、と分けて指定する
- 第一引数に指定する関数は、クラスなどに属さない普通の関数、あるいはstaticメソッドでなくてはならない(→ だったはずが、Dart 2.15でのIsolate/compute改善でこの制約がなくなったように見える。要確認。)
- 第二引数に指定するQ型のmessageおよびR型の戻り値は、プリミティブ型かその組み合わせの単純なクラスである必要がある(上では List<User> を結果として受け取っていてそれはOK)
- 普通に使う上では「ちょっと制約のあるスレッド」くらいの感覚でも良いと思います。
- これで包めばどんなにヘビーな処理でもメインスレッドをフリーズさせずに済むようになりました。

例えば、次のような2つのコードなどを普通に書くとメインスレッドをブロックすることになって、3秒間操作不能になりますが、
```dart
// sleepベタ書き
sleep(Duration(seconds: 3));
// sleepは、Futureを使ってもスレッドを占有する
await Future(() => sleep(Duration(seconds: 3)));
// (ちなみに、ブロックせずに単に3秒後に処理を継続したい場合はこう書くのが正解)
await Future.delayed(Duration(seconds: 3));
```
compute を使った場合は別のIsolateのループがブロックされるだけでメインスレッドのブロック要因にはならず、スムーズな動作が継続されます。
```dart
await compute(sleep, Duration(seconds: 3));
```
## compute は積極的に利用するべき？
- 数百msなどかかる処理は迷うことなく使うべきだが、
- 上で例に出した15ms程度で完了するようなそこまでヘビーでは無い処理では微妙
- compute を使っているFlutterコードをあまり見かけないなと思って、Dart製アプリのOSSとして有名なroughike/inKinoを調べてみたら、次のようなコメントを見つけました。
>前回試したのは1月でしたが、少なくともJSONのパースは十分に高速だったようです。IIRCでは、メインからバックグラウンドのIsolateへの切り替えのオーバーヘッドがJSONのパースよりも長くかかり、そもそもUIスレッドのもたつきもなかった。
- 調査したところ、JSONのパース処理は充分高速で、Isolateのスイッチングコストの方が大きかった、とのこと
## Isolateのスイッチングコスト
上述のMain Isolate上でパースに15msほどかかったJSONレスポンスを用いてiPhone 7 Plusにて確かめたところ、次のようになりました。
- リリースモードとデバッグモードではパフォーマンスに大きな差が出ることがあるので、
- その調査をするときは必ずリリースモードでの数値を確認することが大事です。
- スイッチングコストは30msくらい
- 空のIsolateで確かめたら15msくらいだったりもしましたが、多分パースに必要なメッセージのやり取り分処理時間が長くなったりするのだろう
- というわけで、この例の場合、どちらがベターか悩ましい
  - Main Isolate上で15msで処理(メインスレッドを多少ブロックさせる可能性あり)
  - 別Isolate使って40msで処理(メインスレッドをブロックさせるリスクはほぼゼロ)
- 個人的には許容できないほどスイッチングコストが高くないと感じたので、常に別Isolateでパースというのもありなやり方だとは思いました。
- (とはいえ、iOSネイティブで計測したらスレッドのスイッチコストは1ms未満〜3ms程度だったので、やはりそれよりはコストがかかるなとは思いましたが。)
## 結論
- 必要に応じて compute 関数で並列処理をすると、基本シングルスレッドのFlutterでもユーザー体験を損ねることなくヘビーな処理をさばける
