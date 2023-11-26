# SliverAppBarの上にWidgetを配置する例
```dart
            SliverAppBar(
                automaticallyImplyLeading: false,
                backgroundColor:
                    Theme.of(context).primaryColor.withOpacity(0.5),
                pinned: true,
                snap: false,
                expandedHeight: 100,
                centerTitle: true,
                flexibleSpace: const FlexibleSpaceBar(
                  //PLAN:細かな位置調整は後
                  background: SafeArea(
                    child: Center(
                      child: Padding(
                        padding: EdgeInsets.all(15),
                        child: Center(child: Text('Add content page')),
                      ),
                    ),
                  ),
                ),
                bottom: PreferredSize(
                  preferredSize: const Size.fromHeight(0),
                  child: TextField(
                    //PLAN:入力履歴はローカル・クラウド両方に保存しておく
                    onChanged: (value) {},
                  ),
                ),
              ),
```