



https://www.kamo-it.org/blog/web_feed/

```dart
  Scaffold(
    //ジェスチャーを検知したいときはラップする
    body: GestureDetector(
      child: NestedScrollView(
        headerSliverBuilder: (context, innerBoxIsScrolled) {
          return [
            SliverAppBar(
              //テーマカラーの透明度を半透明にする
              backgroundColor: Theme.of(context).primaryColor.withOpacity(0.5),
              titleSpacing: 5,
              pinned: true,//pinnedするとスクロールしても固定される
              leading: BackButton(
                onPressed: () {
                  //TODO:do something
                },
              ),
              //アクションに
              actions: [
                IconButton(
                  tooltip: 'Set XXX',
                  onPressed: () {
                    //テーマやサイズを設定できる小ウインドウを表示
                  },
                  icon: const Icon(Icons.format_size),
                ),
              ],
            )
          ];
        },
        body: CustomScrollView(
          //do widget
        ),
      ),
    ),
  );
```




!!! tip Gestureはスクロールビューをラップすれば使える
    ```dart
    Widget build(BuildContext context) {
        return SafeArea(
        child: Scaffold(
            body: GestureDetector(
            onVerticalDragDown: (_) {
                print("show onVerticalDragDown");
            },
    ```



