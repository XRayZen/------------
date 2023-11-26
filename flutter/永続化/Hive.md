# Hive
>https://flutter.gakumon.jp/entry/flutter-data-persistence-hive

>https://docs.hivedb.dev/#/README

>https://zenn.dev/naoya_maeda/articles/b67f53af377395

>https://note-tmk.hatenablog.com/entry/2022/09/23/120246

- Hiveとはいわゆるキー・バリュー・ストア(KVS)型データベース
  - KVSとはキー(鍵)とバリュー(値)の組を保存する仕組みをいいます。
  - KVSはRDBのような複雑な関係性を扱うのには向いていないが、追記型のデータを単純に蓄積していくことに向いている
  - 基本的にはある値に対して鍵を設定してデータを蓄積するのみです。その鍵を指定すると、対応する値が取り出せます。
- SQLLiteやS.Prefと比べてパフォーマンスがいい
## install
下記コマンドを打ちます：
```bash
flutter pub add hive
flutter pub add hive_flutter 
flutter pub add hive_generator
flutter pub add build_runner
```
コマンドラインでやると推奨バージョンが勝手に入ります。
## マップ型などプリミティブ型でのサンプル
```dart
//(1) hive_flutterパッケージをインポート
//importはhive/hive.dartではなくhive_flutterの方
import 'package:hive_flutter/hive_flutter.dart';

const boxName = "aBox";

void main() async {
  //(2) Hiveの初期化
  //上記のhive_flutter.dartをインポートしていないとHive.initFlutter()ができない
  await Hive.initFlutter();

  //(3) 使用するボックスをオープン
  await Hive.openBox(boxName);
  //ボックスはデータの入れ物です。引数で与えている名前はファイル名のようなものだと考えてOKです。
  //複数個オープンして使い分けることが可能です。
  //型を指定してオープンすることも可能で、その型のデータを出し入れできるようになります。
  //型と用途で分けてボックスを用意することが多い
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return  MaterialApp(
      debugShowCheckedModeBanner: false,
      initialRoute: '/',
      home:  Home(),
    );
  }
}

class Home extends StatelessWidget {
  Home({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    //(4) ボックスを取得
    //一度Hive.openBoxでオープンしたボックスはHive.boxで取得可能です。
    //今度は瞬時に行える処理ですのでasync / awaitの必要はありません。
    //最初の初期化処理でHive.openBoxしておいて、ウィジェットでの処理ではHive.boxで取得して使うということで当面はOKです。
    //よほど大きなデータを扱ってオンメモリに一括展開するとまずい場合は必要に応じて対応すること
    //古いデータは古いデータ用のボックスに退避させて、必要なとき以外はオープンしない
    var box = Hive.box(boxName);

    //(5) 値を記録
    //キーは255文字までのアスキー文字列、あるいは符号なし32ビット整数という制約
    //ただ上にも書きましたがキーを使ったクエリを書けないので、実際上キーは自動的に付けられるものに任せていい場面がほとんど
    box.put("Prius", "Hybrid");
    box.put("Yaris", "Normal");

    //(6) 値を取得
    //box内の値はget(キー)で取得可能です。
    //キーに対応する値がないとnullになりますが、defaultValueを設定することで対応する値が存在しない場合にgetが返す値を指定できます。
    String val = box.get("Prius", defaultValue: "No info");

    return Scaffold(
      appBar: AppBar(
        title: const Text("ホーム"),
        child: Text("Prius engine is $val")
      ),
      body: Center(
      )
    );
  }
}
```
### Shared Preferencesとの利点
- SQLなんて面倒くさいよっていう方にぴったりなのがHive
- 自作クラスを簡単に永続化できる
  - 自作クラスを使うにはタイプアダプターを生成する必要がある
## カスタムクラスでの定義例
永続化したいクラスのコードを下記に示します。
```dart
//(1) インポート・パート
import 'package:hive/hive.dart';
//part文はファイルの一部を外に切り出ているということを意味]
//タイプアダプタークラスをジェネレータで生成
//生成前はエラー表示だが無視
part 'car.g.dart';

//(2) Hiveタイプアノテーション HiveObject版については後述
//番号を他のクラスと重ならないように手動で付番していく必要
//typeIdは0から233までの整数」のみが使用可能
@HiveType(typeId : 0)
class Car {
  //(3) Hiveフィールドアノテーション
  //各フィールドにも番号を手動で付番していきます。
  //フィールド番号は0から255の範囲
  //クラスを改変するとき、一度書き出したデータに、
  //HiveFieldの番号やHiveTypeのTypeIdを変えてはいけない
  //クラス定義をアプリ運用中にアップデートする際の注意は下記に記す
  //新しいフィールドについては番号を足していく。
  //削除したフィールドの番号を再利用しない、というのが原則
  @HiveField(0)
  String name;

  @HiveField(1)
  String engine;

  Car(this.name, this.engine);

  @override
  String toString(){
    return name+" "+engine;
  }
}
```
### タイプジェネレータの生成
Hiveに記録するクラスの定義を終えたところで、ターミナルを起動して
```
flutter packages pub run build_runner build
```
というコマンドを打ちます。
コマンドを実行すると先程のpart文で書いていた「cart.g.dart」が生成されます。

!!! attention クラスを更新してフィールドを追加した場合などは、そのたびに冒頭のコマンドラインでタイプジェネレータクラスを更新する必要があります。
### クラスのアップデート対応
!!! attention 但し、エラーを回避する為にフィールドをアップデートしたときは極力前のデータは消しておくべき
アプリをアップデートする際、クラスを更新して新しい情報を付加することはよくあると思います。このときは下記ルールに従う必要があります：
- フィールドの番号を変えない
- フィールドの名前やpublic/privateの変更はフィールド番号を変えない限り影響なし
- フィールドを削除する場合、そのフィールド番号を再利用しない
- フィールドの型の変更はできない

要するにフィールドの番号を変えてはいけないということ
Hiveのデータのなかではフィールドの名前は無視されていて番号だけで管理されています。

フィールドの型は変更できないため、そうしたい場合は新しいフィールド番号を付番したフィールドを作って移行する必要があります。
- 新しいフィールドを追加しても、古いアダプタで書き込まれたデータを読み込むことは可能。
- 新しいアダプタで書き込まれたデータを古いアダプタで読み込むことは可能。

古いアダプタには存在していない新しいフィールドはすべて無視されるのみ
なのでフィールド番号をキープしている限り、クラス定義とアダプタをアップデートしても、データが壊れてしまう、読めなくなるということはありません。


## カスタムクラスでの読み書き
```dart
import 'package:flutter/material.dart';
import 'package:hive_flutter/hive_flutter.dart';
import 'car.dart';

const carBoxName = "carBox";
<!-- 省略 アプリエントリーポイントでの使用 -->
await Hive.initFlutter();
//(1) タイプアダプタを登録
Hive.registerAdapter(CarAdapter());
//(2) 型指定してボックスをオープン
await Hive.openBox<Car>(carBoxName);
<!-- ここで注意なのですが、タイプアダプタを登録するのはopenBoxの前です。
openBoxという響きのニュアンスからこちらが前のような直感があるかもしれませんが、ここは注意です。
openするためにタイプアダプタを使うので、順番を間違ってはいけません。 -->
<!-- 中略 -->
<!-- Widget内で使用 -->
//(3) 型指定でboxを取得
//ボックスを使用する際、こちらも型を指定
var box = Hive.box<Car>(carBoxName);

//(4) 値を記録
//前出のものと同様キーと対になる値を組みにしてputします。まとめてputAllも可能です。
box.put("car1", Car("Prius", "Hybrid"));
box.put("car2", Car("Yaris", "Normal"));

//(5) 値を取得
//getはキーに対応する値が存在しないときはnullを返すので、ここではCar?型を返すと決まっています。ですのでCar?と宣言
Car? val = box.get("car1", defaultValue: Car("No name", "No engine"));
//この書き方がベストか
Car val = box.get("car1") ?? Car("No name", "No engine");
```
## HiveObject
>https://flutter.gakumon.jp/entry/flutter-hive-hiveobject-hivelist
- 上記のやり方でも基本的にデータの永続化はできるがキーの扱いが面倒
- Hiveではデータを選別するクエリについてはサポートしない方針
- なのでキーは単に値を引っ張るための合言葉になっています。
  - となると、キー周りを自分のオブジェクトに持たせて、各インスタンス自身にキーを管理させて外からは隠蔽するHiveデータ用のクラスを作ろう、という発想になる
- それがHiveObjectです。
  - 仕組みは単純で、各インスタンスが所属するボックスとキーを自分で持っておく形にして、そのインスタンス単独で保存や削除をできるようにしているものです。

HiveObjectを使うには、データを表すクラスでHiveObjectを継承します：
```dart
import 'package:hive/hive.dart';
part 'car.g.dart';

@HiveType(typeId : 0)
//(1) HiveObjectを継承
class Car extends HiveObject{
  @HiveField(0)
  String name;

  @HiveField(1)
  String engine;

  Car(this.name, this.engine);

  @override
  String toString(){
    return name+" "+engine;
  }
}
```
HiveObjectを継承している以外は、上記のアノテーションを付与しているだけの通常のHive用オブジェクト定義です。

HiveObjectの主な属性値とメソッドは以下の通りです。
継承すると、これらが使えるようになります：
- box
  - そのデータが所属するbox
- isInBox
  - そのデータがなんらかのboxにストアされているなら真
- key
  - そのデータに対応するキー
- delete()
  - そのデータをboxから削除する
- save()
  - そのデータをboxにストアする

ほぼ、delete()とsave()を抑えておけばOKです。

具体的な使い方の例を次に示す：
```dart
<!-- エントリーポイント内 -->
await Hive.initFlutter();
Hive.registerAdapter(CarAdapter());
await Hive.openBox<Car>(carBoxName);
<!-- Widget内で使用 -->
var box = Hive.box<Car>(carBoxName);
//(1) 値を用意
var car1 = Car("Prius", "Hybrid");
var car2 = Car("Yaris", "Normal");
var car3 = Car("Aqua", "Hybrid");
for(Car c in box.values){c.delete();}
//ボックスに入れることでHiveObjectインスタンスにボックスの情報が格納され、
//addやdeleteができるようになります。
box.addAll([car1, car2, car3]);
//(2) box内部の値を全て取得
//一旦carsにリストとしてboxの全値を入れています。
//「...」はスプレッド演算子で、こういうときに便利です。
var cars = <Car>[...box.values];
//(3) 値の更新
//cars[2](carsの3番目の要素)のengineを書き換え、saveを呼び出して永続化しています。
//このようにboxやキーを意識しないでデータを永続化できるのが利点
cars[2].engine = "Special Engine";
cars[2].save();
//(4) 値の削除
//cars[2](carsの3番目の要素)のengineを書き換え、saveを呼び出して永続化しています。
//このようにboxやキーを意識しないでデータを永続化できるのが利点
cars[1].delete();
```
- HiveObjectの基本の使い方は以上の通り
- boxに所属させたら、あとはsaveやdeleteを単体で行うことができ、キー自体を陽に扱う必要のない仕組み、という捉え方でよい
typeIdは0でPartクラスのtypeIdと重ならないように設定します。コピペでデータクラスの実装の雛形を作るとき、typeIdの設定には特に注意
## HiveList
これはインスタンスに別のインスタンスを持たせるときに便利
いわゆるリレーション
HiveObjectのインスタンスに別クラスのインスタンスの参照を持っていて、それを永続化できる
```dart
import 'part.dart';
import 'package:hive/hive.dart';

part 'car.g.dart';

//typeIdは0でPartクラスのtypeIdと重ならないように設定します。
//コピペでデータクラスの実装の雛形を作るとき、typeIdの設定には特に注意
@HiveType(typeId : 0)
//(1) HiveObjectの継承
class Car extends HiveObject{
  @HiveField(0)
  String name;

  @HiveField(1)
  String engine;

  //(2) HiveListを定義しています
  //リスト要素を持たせるためHiveList型の属性を定義
  //HiveListのコンストラクタはBoxを必要とするため、
  //Boxがオープンされてからの初期化になります。
  //なのでこの場で空のHiveListで初期化することができません。
  @HiveField(2)
  HiveList<Part> parts;
  //なのでコンストラクタの方で、初期化することを宣言しておきます。
  Car(this.name, this.engine, this.parts);

  @override
  String toString(){
    var s = StringBuffer();
    for(Part p in parts){
      s.write(p.toString()+" ");
    }
    return name+" "+engine+" 部品["+s.toString()+"]";
  }
}
```
HiveListのPartクラスを定義します。
HiveListにいれる要素はHiveObjectを継承している必要がある
ファイルは「lib/part.dart」としておきます。
これはtypeIdを1にしているくらいであとはシンプルなもの：
```dart
import 'package:hive/hive.dart';
part 'part.g.dart';

@HiveType(typeId : 1)
class Part extends HiveObject{
  @HiveField(0)
  String name;

  @HiveField(1)
  String status;

  Part(this.name, this.status);

  @override
  String toString(){
    return "("+name+"・"+status+")";
  }
}
```
- 使用例
```dart
<!-- エントリーポイント内 -->
//(1) Hive初期化・アダプタ登録・Boxオープン (順番注意)
//openBoxの前にアダプタを登録する必要があるので注意
await Hive.initFlutter();

Hive.registerAdapter(CarAdapter());
Hive.registerAdapter(PartAdapter());

await Hive.openBox<Car>(carBoxName);
await Hive.openBox<Part>(partBoxName);
<!-- Widget内で使用 -->
    //(2) 書き込むデータを準備
    var partBox = Hive.box<Part>(partBoxName);
    var part1 = Part("タイヤ", "新品");
    var part2 = Part("ハンドル", "中古");
    var part3 = Part("ドア", "新品");
    var part4 = Part("ステレオ", "故障");

    //(3) 繰り返し実行したときに増えないようにすでにBox内部にあるデータを消去
    for (var element in partBox.values) {
      element.delete();
    }
    partBox.addAll([part1, part2, part3, part4]);

    var carBox = Hive.box<Car>(carBoxName);
    var car1 = Car("Prius", "Hybrid", HiveList(partBox));
    var car2 = Car("Yaris", "Normal", HiveList(partBox));
    var car3 = Car("Aqua ", "Hybrid", HiveList(partBox));

    car1.parts.addAll([part1, part2, part4]);
    car2.parts.addAll([part1, part2, part3]);
    car3.parts.addAll([part1]);

    for (var element in carBox.values) {
      element.delete();
    }
    carBox.addAll([car1, car2, car3]);
    //(4) Car, PartのBoxを取得、値をリストに入れる
    var carBox = Hive.box<Car>(carBoxName);
    var cars = <Car>[...carBox.values];

    var partBox = Hive.box<Part>(partBoxName);
    var parts = <Part>[...partBox.values];

    //(5) 画面に表示する文字列の作成
    var texts = <String>[];
    texts.add("【読み込んだデータ】");
    for (var car in cars) {
      texts.add(car.toString());
    }
    texts.add("\n【加工したデータ】");

    //(6) 最初の車の最初のPartsのstatusを変えてみる
    if (cars.length > 1) {
      cars[0].parts[0].status = "リコール";
    }
    //(7) 「ハンドル」という名前のPartsを削除してみる
    parts.where((e) => e.name == "ハンドル").forEach((e) {
      e.delete();
    });

    //加工後のCarのデータを出力文字列に入れる
    for (var car in cars) {
      texts.add(car.toString());
    }
```





