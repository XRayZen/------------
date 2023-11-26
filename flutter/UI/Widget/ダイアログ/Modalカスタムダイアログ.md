# ModalRouteを使ったカスタムダイアログの実装
>https://www.egao-inc.co.jp/programming/flutter_custom_dialog/

![](https://www.egao-inc.co.jp/wp-content/uploads/2019/11/f6d88e7aabecee465e171660a7bbadc5.png)
## 説明
ModalRouteを継承したページを用意し、モーダル画面遷移させて現在のページの上にかさねる感じになります。
よって、現在のページ上にボタンがあったとしてもタッチは届かず、ダイアログとしての機能を満たします。
## ModalRouteを継承したルートクラスを実装
ModalRouteを継承したModalOverlayクラスを追加します。
```dart
/*
 * モーダルオーバーレイ
 */
class ModalOverlay extends ModalRoute<void> {
 
  // ダイアログ内のWidget
  final Widget contents;
 
  // Androidのバックボタンを有効にするか
  final bool isAndroidBackEnable;
 
  ModalOverlay(this.contents, {this.isAndroidBackEnable = true}) : super();
 
  @override
  Duration get transitionDuration => Duration(milliseconds: 100);
  @override
  bool get opaque => false;
  @override
  bool get barrierDismissible => false;
  @override
  Color get barrierColor => Colors.black.withOpacity(0.5);
  @override
  String get barrierLabel => null;
  @override
  bool get maintainState => true;
 
 
  @override
  Widget buildPage(
      BuildContext context,
      Animation<double> animation,
      Animation<double> secondaryAnimation,
      ) {
    return Material(
      type: MaterialType.transparency,
      child: SafeArea(
        child: _buildOverlayContent(context),
      ),
    );
  }
 
  @override
  Widget buildTransitions(BuildContext context, Animation<double> animation, Animation<double> secondaryAnimation, Widget child) {
    return FadeTransition(
      opacity: animation,
      child: ScaleTransition(
        scale: animation,
        child: child,
      ),
    );
  }
 
  Widget _buildOverlayContent(BuildContext context) {
    return Center(
      child: dialogContent(context),
    );
  }
 
  Widget dialogContent(BuildContext context) {
    return WillPopScope(
      child: this.contents,
      onWillPop: () {
        return Future(() => isAndroidBackEnable);
      },
    ); 
  }
}
```
Androidで実機のバックボタンを押すとダイアログが閉じてしまうので、有効無効の切り替えができるようにしました。dialogContentメソッドのWillPopScopeクラスのonWillPopでポップする際にコールバックが呼ばれて、bool値を返すことで切り替えしてます。

また、ModalOverlayではcontentsというWidgetをもらってそれを表示させる
## ダイアログのコンテンツクラスを実装
- CustomDialogというクラスを作り、ここでModalOverlayのコンテンツを作って渡すのと、push遷移
```dart
class CustomDialog {
 
  BuildContext context;
  CustomDialog(this.context) : super();
 
  /*
   * 表示
   */
  void showCustomDialog() {
 
    Navigator.push(
      context,
      ModalOverlay(
        Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
 
              Stack(
                children: <Widget>[
 
                  Image.asset(
                    "images/fukidashi.png",
                    fit: BoxFit.fitWidth,
                  ),
 
                  Container(
                      height: 200.0,
                      child: Center(
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: <Widget>[
 
                            /*
                             * タイトル
                             */
                            Text(
                              "カスタムダイアログ",
                              style: TextStyle(
                                fontSize: 30.0,
                                fontWeight: FontWeight.bold,
                                locale: Locale("ja", "JP"),
                              ),
                            ),
 
                            /*
                             * メッセージ
                             */
                            Text(
                              "こんな感じでダイアログが出せるよ",
                              style: TextStyle(
                                fontSize: 16.0,
                                locale: Locale("ja", "JP"),
                              ),
                            ),
                          ],
                        ),
                      )
                  )
                ],
              ),
 
              /*
               * OKボタン
               */
              Row(
                children: <Widget>[
                  Expanded(
                    child: Container(),
                  ),
 
                  FlatButton(
                    color: Colors.blue,
                    child: Text(
                      "OK",
                      style: TextStyle(
                        color: Colors.white,
                        fontSize: 18.0,
                        fontWeight: FontWeight.bold,
                        locale: Locale("ja", "JP"),
                      ),
                    ),
                    onPressed: () {
 
                      hideCustomDialog();
                    },
                  ),
 
                  SizedBox(
                    width: 80.0,
                  )
                ],
              ),
            ],
          )
        ),
        isAndroidBackEnable: false,
      ),
    );
  }
 
 
  /*
   * 非表示
   */
  void hideCustomDialog() {
 
    Navigator.of(context).pop();
  }
}
```
## 呼び出し
```dart
onPressed: () {
 
   CustomDialog(
      context,
   ).showCustomDialog();
},
```