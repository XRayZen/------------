>https://zenn.dev/inari_sushio/articles/98a094b4faa6cc
# Route とは
全体像で分かる通り、全てのルートを包括する抽象ルートです。
Navigator と連携して「アプリに表示する画面」を決定するための、様々なプロパティやメソッドをインタフェースに持ちます。これらをひも解くと、少しルートの正体が見えてくると思います。

例えば...
- navigator: NavigatorState?
  - ... このルートがどの Navigator に属しているか。
- isActive: bool
  - ... このルートが Navigator に属しているか否か。
- isCurrent: bool
  - ... このルートが Navigator が管理するルートエントリーのリストの一番上にあるか（ ≒ 現在画面に表示されているルートか）否か。
- overlayEntries: List<OverlayEntry>
  - ... このルートにひも付く OverlayEntry のリストを規定。

!!! 【 OverlayEntry とは？ 】
    画面を構成する要素の一つであり、ウィジェットビルダーを持つ ChangeNotifier である。Overlay ウィジェットによりリストで管理され、ウィジェットビルダーが呼び出される。opaque（画面全体を覆うか否か）、maintainState（さらに上層に opaque な OverlayEntry があった場合にツリーに残すか否か）などの状態を持つ。

!
【 Overlay とは？ 】
Navigator によりウィジェットツリーに挿入される StatefulWidget であり、日本語で「上に被せる物」という意味。OverlayEntry のリストを状態に持つ。Overlay.of(context) で OverlayState を取得することができ、OverlayEntry を追加したり並び替えたりすることができる。

settings: RouteSettings
... このルートの RouteSettings。値の型が RouteSettings の場合は「ページレスルート」と呼ばれる一方、型が RouteSettings を継承する Page の場合は「ページベースルート」と呼ばれる（参考）。
!
【 RouteSettings とは？ 】
そのルートにひも付く関連データ。ルートの名称（ '/' とか）、ルートに渡す引数などの情報。

!
【 Page とは？ 】
RouteSettings を継承し、キーや復元IDなどの追加情報を持つ。Navigator 2.0 の場合は Navigator により Page.createRoute() が呼び出され、ページからルートが生成される。その場合の Route.settings は Page になる。

install(): void
... このルートが Navigator に追加される時に呼び出される。Route.overlayEntries リストを生成するために使われ、このメソッド呼び出し後はリストに OverlayEntry が最低一つは存在しなくてはならない。
didPop(): bool
... このルートをポップするリクエストが発生すると呼び出される。true を返すことで、Navigator はルートエントリーのリストからこのルートを除き、様々なメソッドを経て遷移アニメーションが完了した後に Route.navigator に null を代入する。
💬 どうやら Route とはウィジェットを生成してくれる OverlayEntry を複数持ち、Navigator と連携するための様々な設定情報や状態を持つもの、




## 4. ModalRoute とは
TransitionRoute を継承する抽象クラスで、画面全体を覆って、その下層ルートへのユーザーによるアクションをシャットダウンする「モーダルバリア（modal barrier）」と呼ばれる OverlayEntry（もしくはウィジェット）を生成するための設定やメソッドが追加されている。

また、このクラスから buildPage(): Widget メソッドが規定されており、このメソッドを元に「モーダルバリア」とは別の「モーダルスコープ（modal scope）」と呼ばれる OverlayEntry（もしくはウィジェット）が生成される。

![](https://storage.googleapis.com/zenn-user-upload/9e3677b42128-20220101.png)

このモーダルバリアは設定により Color や ImageFilter を付けることが可能であるが、未設定の場合は透明となる。つまり、「画面全体を覆う」とはいえ、必ずしもそれが不透明（opaque）とは限らず、ページを表示するルートにもダイアログを表示するルートにもなり得る。

また、モーダルバリア部分をタッチ等することで、そのルートをポップすることができるか否かを設定する barrierDismissible: bool プロパティもこのクラスで規定されている。

!
【 モーダルとは？ 】
「閉じるまで他のことはさせねーぜ」

そして、モーダルスコープ部分の表示・非表示を決めているのが offstage: bool プロパティである。Navigator にルートが追加されると createOverlayEntries() が ModalRoute 内部で呼び出されてモーダルスコープが作られるが、このウィジェットで使われている Offstage ウィジェットにこのルートの offstage 値があてがわれ、実際の表示・非表示が行われる。

!
【 Offstage とは？ 】
その child の表示・非表示が offstage プロパティの値により決まるウィジェット。offstage が true の時に非表示になるが、Visibility ウィジェットと異なりツリー構造は false の時と変わらない。child を描画せずヒットテストの対象にもならないが、child にフォーカスノードがあればフォーカスは可能（キー入力は受け付ける）である。

ちなみに ModalRoute は ModalRoute.of(context) により、その BuildContext （エレメント）が属するルートを取得することができる。これにより、ルートに渡されている引数などの情報を取得することができる。




















