
>https://www.kodeco.com/14214369-infinite-scrolling-pagination-in-flutter
- 無限スクロールのテクニックをマスターしてください。
  - 無限スクロールページネーション
  - エンドレススクロールページネーション
  - オートページネーション
  - レイジーローディングページネーション
  - 連続スクロールページネーション
  - プログレッシブローディングページネーション
- など、さまざまな名前がついています。

このチュートリアルが終わるころには、あなたは知っているでしょう。
- 無限スクロールパジネーションとは何か
- その代替案は何ですか？
- なぜそれがそんなに重要なのか
- ページングされていないアプリからページングされたアプリに移行する方法
- その知識をどのようなプロジェクトでも活用する方法

# はじめに

## ページ送り(Pagination)を理解する
- 例としてバー来た客をユーザーと呼び、バーがアプリ
- バーテンダーであるあなたは、このユーザーがたくさんのデータ・ドリンクを消費することを知っています。
  - しかし、あなたはまだ、その数を知りません。
- あなたは、最高のカスタマー・エクスペリエンスを提供することに専念しているので、ユーザーにサービスを提供する最適な方法は何かと考え始めます。
- そして、近くにあったナプキンを手に取り、選択肢を書き出し始めます
- 古き良きウェブ的なページネーションが遅延読み込みページ送りのいい例
Google検索からのページ選択バー
![](../../images/regular-pagination.png)
- あるページを見終わるたびに、手動で次のページを求めなければならない。
- 良いことだが、まだ完璧ではない
- 自動化できればもっといいのだが
# Automating Pagination
- このプロセスは、無限スクロールのページネーションと同じです。
- ユーザーが画面をスクロールしていくと、その様子を見ながら、ユーザーが画面の下に到達する前にさらにデータを取得し、無限リストをスクロールしているようなスムーズな体験をユーザーに提供します。
- ほら、クールな子供向けアプリはみんなやっていることです。
![](../../images/infinite-scroll-showcase.gif)

## ページ分割された ArticleListView を作成する
lib/ui/list の中に paged_article_list_view.dart という新しいファイルを作成することから始めましょう。このファイルの中に、以下の内容を記述してください。
```dart
import 'package:flutter/material.dart';
import 'package:readwenderlich/data/repository.dart';
import 'package:readwenderlich/entities/article.dart';
import 'package:readwenderlich/ui/exception_indicators/empty_list_indicator.dart';
import 'package:readwenderlich/ui/exception_indicators/error_indicator.dart';
import 'package:readwenderlich/ui/list/article_list_item.dart';
import 'package:readwenderlich/ui/preferences/list_preferences.dart';

//これは画面上の古いページ分割されていないリストを置き換えるウィジェットです。
//これは ArticleListView のページングされたバージョンです。
class PagedArticleListView extends StatefulWidget {
  const PagedArticleListView({
    //囲んだウィジェットから Repository インスタンスを要求しています。
    //Repository はスタータープロジェクトに付属しており、リモート API との通信をサポートします。
    @required this.repository,
    //同様に、ListPreferences のインスタンスを、囲んでいるウィジェットに問い合わせる
    this.listPreferences,
    Key key,
  })  : assert(repository != null),
        super(key: key);

  final Repository repository;
  final ListPreferences listPreferences;

  @override
  _PagedArticleListViewState createState() => _PagedArticleListViewState();
}

class _PagedArticleListViewState extends State<PagedArticleListView> {
  //前のステップで受け取った ListPreferences に、毎回 widget.listPreferences と入力しなくても
  //アクセスできるようにするために、計算されたプロパティを作成しているのです。
  ListPreferences get _listPreferences => widget.listPreferences;
 
  // TODO: Instantiate a PagingController.

  @override
  //ページ分割されたリスト・ウィジェットをまだ理解していない間は、
  //プレースホルダーは巨大なXで画面を埋めます。このことはすぐにでも理解できるはずです。
  Widget build(BuildContext context) => const Placeholder();
}

```
1. これは、画面上の古いページ分割されていないリストを置き換えるウィジェットです。これは ArticleListView のページングされたバージョンです。
2. ウィジェットを囲んで、Repository インスタンスを要求しています。Repository はスタータープロジェクトに付属しており、リモート API との通信をサポートします。
3. 同様に、今度は ListPreferences インスタンスを付属のウィジェットに要求しています。
4. 前のステップで受け取った ListPreferences にアクセスするために、毎回 widget.listPreferences と入力する必要がないように、計算されたプロパティを作成しています。
5. ページ分割されたリスト・ウィジェットをまだ理解していない間は、プレースホルダが巨大な X で画面を埋めます。このことは、すぐにでも説明します。

さて、今作成したものを視覚化する必要があります。

## リストウィジェットの交換
古いArticleListViewを新しいPagedArticleListViewに交換します。そのためには、lib/ui/list/article_list_screen.dart を開き、新しいファイルへの import を一番上に追加してください。
すでにインポートを使用しているので、まもなく使用されなくなるArticleListViewインポートを削除して、いくつかの掃除をする機会を得ます。
```dart
import 'package:readwenderlich/ui/list/article_list_view.dart';
```
build()にジャンプして、Scaffoldのbodyプロパティを置き換えます。
```dart
body: PagedArticleListView(
  repository: Provider.of<Repository>(context),
  listPreferences: _listPreferences,
),
```
このプロジェクトの依存性注入システムであるプロバイダから、リポジトリのインスタンスを取得しています。

これで完了です。
lib/ui/list/article_list_view.dart を削除することができます。

ビルドして実行します。プレースホルダーが動作しているのが見えるはずです。
![](../../images/placeholder.png)
## 無限スクロールページ処理(プルトゥフェッチ)
上記のドリンクサービスの場面では、製品の観点から無限スクロールのページネーションに注目しました。今度は、開発者モードになり、目標を分割し、それを解決するために必要なことを検討してみましょう。
- ユーザーのスクロール位置を監視し、他のページを事前に取得できるようにする。
- ユーザーのスクロール位置を把握して、他のページを事前に取得できるようにする。
![](../../images/flow-diagram.png)
- 各ステータスごとにインジケータを表示することで、ユーザーに常に情報を提供します
![](../../images/pagination-status-indicators.png) 
- 異なる画面、場合によっては他のレイアウトを使用して再利用可能なソリューションを作成します。その一例がグリッドです。理想的には、このソリューションは、他の状態管理アプローチを持つ異なるプロジェクトにも移植可能であるべきです。

大変そう？そんなことはありません。これらの問題は、`Infinite Scroll Pagination` パッケージですでに解決されています。次のセクションでは、このパッケージについて詳しく見ていきます。
# infinite_scroll_paginationパッケージについて
pubspec.yamlを開き、次のように置き換える
```dart
dependencies:
  flutter:
    sdk: flutter
  infinite_scroll_pagination: ^3.1.0
```
画面上部のFlutterコマンドバーのPub getをクリックして、最新の依存関係をダウンロードします。

`Infinite Scroll Pagination`パッケージは、あなたの仕事を、赤ちゃんからお菓子を盗んだり、樽の中の魚を撃ったり、3ピースのジグソーパズルを組み立てるのと同じくらい簡単にしてくれます。

## まずは、`PagingController`を定義する
そして、これが最初のピース`PagingController`
![](../../images/first-puzzle-piece.png)

- PagingControllerは、ページングされたウィジェットのコントローラ
- ページ分割の現在の状態を保持し、必要なときにリスナーからページを要求する役割を担う

FlutterのTextEditingControllerやScrollControllerなどを扱ったことがある人なら、PagingControllerはなじみやすいと思います。
### PagingController のインスタンス化
lib/ui/list/paged_article_list_view.dart に戻り、ファイルの一番上に新しいライブラリへの import を追加します。
```dart
import 'package:infinite_scroll_pagination/infinite_scroll_pagination.dart';
```
ここで、 // TODO: PagingController のインスタンスを作成する に置き換えてください。
```dart
// 1 インスタンスを作成する際には、2 つの一般的な型を指定する必要
//`int`: これは、エンドポイントがページを識別するために使用するタイプです。 
//今回使う`API`の場合、ページ番号です。他の API では、違うタイプが使われます。
//このページ分割戦略の多様性から、このパッケージではこれらの識別子を **page keys** と呼ぶ
//`Article`:リスト項目をモデル化する型
final _pagingController = PagingController<int, Article>(
  // 2 firstPageKey パラメータを使って、ページ初期値を設定する必要がある
  //今回使う`API`の場合、ページキーは1から始まりますが、他のAPIの場合は0から始まるかもしれない
  firstPageKey: 1,
);

@override
void initState() {
  // 3 このようにして、新しいページを要求されたら処理をリッスンするために関数を登録する
  _pagingController.addPageRequestListener((pageKey) {
    _fetchPage(pageKey);
  });
  super.initState();
}

Future<void> _fetchPage(int pageKey) async {
  // TODO: Implement the function's body.
  //ページを要求されたら取得処理をするコードを書く
}

@override
void dispose() {
  // 4 コントローラを dispose() するのを忘れないように
  _pagingController.dispose();
  super.dispose();
}
```
上のコードが何をするのか、順を追って説明します。
1. `PagingController` のインスタンスを作成する際には、2 つの一般的な型を指定する必要があります。あなたのコードでは、それらは
   1. `int`: これは、エンドポイントがページを識別するために使用するタイプです。 今回使う`API`の場合、ページ番号です。他の API では、違うタイプが使われます。このページ分割戦略の多様性から、このパッケージではこれらの識別子を **page keys** と呼ぶ
   2. `Article`:リスト項目をモデル化する型
2. firstPageKey パラメータを使って、ページ初期値を設定する必要がある
   - 今回使う`API`の場合、ページキーは 1 から始まりますが、他の API の場合は 0 から始まるかもしれない
3. このようにして、新しいページを要求されたら処理をリッスンするために関数を登録する
4. コントローラを dispose() するのを忘れないように
### ページの取得
`_fetchPage()`の実装をこれに置き換えること
```dart
Future<void> _fetchPage(int pageKey) async {
  try {
    final newPage = await widget.repository.getArticleListPage(
      number: pageKey,
      size: 8,
      // 1 現在のフィルタリングとソートのオプションをリポジトリに転送している
      filteredPlatformIds: _listPreferences?.filteredPlatformIds,
      filteredDifficulties: _listPreferences?.filteredDifficulties,
      filteredCategoryIds: _listPreferences?.filteredCategoryIds,
      sortMethod: _listPreferences?.sortMethod,
    );

    final previouslyFetchedItemsCount =
        // 2 itemListは、PagingControllerのプロパティです。
        //これまでに読み込まれたすべてのアイテムを保持します。
        //itemListの初期値はnullなので、?条件付きプロパティアクセスを使用しています。
        _pagingController.itemList?.length ?? 0;

    final isLastPage = newPage.isLastPage(previouslyFetchedItemsCount);
    final newItems = newPage.itemList;

    if (isLastPage) {
      // 3 新しい項目を作成したら、`appendPage()`あるいは`appendLastPage()`をコールしてそれをコントローラに知らせます。
      _pagingController.appendLastPage(newItems);
    } else {
      final nextPageKey = pageKey + 1;
      _pagingController.appendPage(newItems, nextPageKey);
    }
  } catch (error) {
    // 4 エラーが発生した場合は、コントローラのerrorプロパティにその値を指定します。
    _pagingController.error = error;
  }
}
```
1. 現在のフィルタリングとソートのオプションをリポジトリに転送している
2. itemListは、PagingControllerのプロパティです。これまでに読み込まれたすべてのアイテムを保持します。itemListの初期値はnullなので、?条件付きプロパティアクセスを使用しています。
3. 新しい項目を作成したら、`appendPage()`あるいは`appendLastPage()`をコールしてそれをコントローラに知らせます。
4. エラーが発生した場合は、コントローラのerrorプロパティにその値を指定します。

ビルドして実行し、エラーが発生していないことを確認します。視覚的、機能的な変化は期待しないでください。
## PaginatedListViewの使い方
build()に移る前に知っておいてほしいことがあります。
![](../../images/second-puzzle-piece.png)
2つ目のピースは、その名前が示すとおり、通常のListViewのページ分割バージョンです。
そして図が示すように、そこにあなたのコントローラを収めることになります。

lib/ui/list/paged_article_list_view.dart で、古い build() を置き換えてください。
```dart
@override
Widget build(BuildContext context) =>
    // 1 FlutterのRefreshIndicatorでスクロール可能なウィジェットをラッピングすると、
    //swipe to refreshと呼ばれる機能が使えるようになります。
    //ユーザーはこれを利用して、リストを上から下に引っ張って更新することができます。
    RefreshIndicator(
      onRefresh: () => Future.sync(
        // 2 PagingController は refresh() というデータを更新する関数を定義している
        //refresh() の呼び出しを Future でラップしているのは、 
        //RefreshIndicator の onRefresh パラメータがそれを想定しているから
        () => _pagingController.refresh(),
      ),
      // 3 PagedListView には、リストアイテムの間にセパレータを追加するための 
      //separated() コンストラクタが用意されています。
      child: PagedListView.separated(
        // 4 パズルのピース(pagingController)をつなげる
        pagingController: _pagingController,
        padding: const EdgeInsets.all(16),
        separatorBuilder: (context, index) => const SizedBox(
          height: 16,
        ),
      ),
    );
```
Here’s what’s going on:
1. FlutterのRefreshIndicatorでスクロール可能なウィジェットをラッピングすると、swipe to refreshと呼ばれる機能が使えるようになります。ユーザーはこれを利用して、リストを上から下に引っ張って更新することができます。
2. PagingController は refresh() というデータを更新する関数を定義しています。refresh() の呼び出しを Future でラップしているのは、 RefreshIndicator の onRefresh パラメータがそれを想定しているから
3. PagedListView には、リストアイテムの間にセパレータを追加するための separated() コンストラクタが用意されています。
4. パズルのピース(pagingController)をつなげていく
## リストアイテムの作成するためにBuilder Delegateを作成
リストアイテムを構築するためにBuilder Delegateを作成する
- 3つ目のピースを埋める
![](../../images/third-puzzle-piece.png)

最後の仕上げとして、`PagedListView`にこの新しいパラメータを`pagingController`、`padding`、`separatorBuilder`と同じレベルで指定して、コード内の同じギャップを埋めます。
```dart
builderDelegate: PagedChildBuilderDelegate<Article>(
  itemBuilder: (context, article, index) => ArticleListItem(
    article: article,
  ),
  firstPageErrorIndicatorBuilder: (context) => ErrorIndicator(
    error: _pagingController.error,
    onTryAgain: () => _pagingController.refresh(),
  ),
  noItemsFoundIndicatorBuilder: (context) => EmptyListIndicator(),
),
```
`PagedChildBuilderDelegate`は、無限スクロールのページネーションに関わるすべてのウィジェットのためのビルダーのコレクションである。
このコードでは3つのパラメータしか指定していないが、7つのパラメータをすべて知っておくと、将来的に役に立つかもしれない。
- `itemBuilder`: これはリスト項目を構築するものです。これは唯一の必須パラメータで、他のパラメータはすべてデフォルトです。
- `firstPageErrorIndicatorBuilder`: これは、最初のページの取得にエラーが発生したことをユーザーに通知するウィジェットを構築するものです。このシナリオでは、リストアイテムはまだロードされていないので、このウィジェットがスペース全体を埋めるようにします。インジケータに再試行ボタンを追加する場合は、コントローラで `refresh()` を呼び出すことを確認してください。
- `newPageErrorIndicatorBuilder`: これもまたエラーを示しますが、それ以降のページ要求に対してです。すでにいくつかのアイテムがロードされているので、ここで作成したウィジェットはリストの一番下に表示されます。インジケータに再試行ボタンを追加する場合は、ユーザーがボタンをタップしたときに、コントローラで `retryLastRequest()` を呼び出すことを確認してください。
- `firstPageProgressIndicatorBuilder`: これは、最初のページをロードしている間に表示されるウィジェットを構築します。まだアイテムはロードされていないので、このウィジェットがスペース全体を埋めるようにします。
- `newPageProgressIndicatorBuilder`: これは前のビルダーと似ていますが、後続のページをロードしている間に表示されます。すでにいくつかのアイテムがロードされているため、ここで構築したウィジェットはリストの一番下に表示されます。
- `noItemsFoundIndicatorBuilder`: APIがうまくいって空のリストを返した場合はどうでしょうか？技術的にはこれはエラーではなく、通常は選択されたフィルタオプションが多すぎることに関連しています。ここで作成したウィジェットは、この「0件」のシナリオをカバーしています。
- `noMoreItemsIndicatorBuilder`: ここでは、ユーザーがリストの最後に到達したときに表示するウィジェットをオプションで作成します。

ビルドして実行します。下のGIFに示すように、デバイスの接続をオフにして、スワイプで更新してみてください。これは、`RefreshIndicator`とカスタム エラー インジケータ ウィジェットの両方をテストするのに役立ちます。
![](../../images/filters-not-working.gif)

!!! question アプリにバグがある どうやら最終仕上げではなかったようです。何が問題なのか
!!! bug フィルターをかけても効果が出ていない
## フィルタの適用
!!! tip 初見で動いた場合は疑ってかかれとよく言われる

バグの原因はここにあります。ユーザがフィルタを適用すると、ArticleListScreen は PagedArticleListView を全く新しい ListPreferences で再構築しますが、誰もその変更について PagingController に警告を出しません。

これを解決するための場所は lib/ui/list/paged_article_list_view.dart です。_PagedArticleListViewStateの中に、以下の関数オーバーライドを追加してください。
```dart
@override
void didUpdateWidget(PagedArticleListView oldWidget) {
  if (oldWidget.listPreferences != widget.listPreferences) {
    _pagingController.refresh();
  }
  super.didUpdateWidget(oldWidget);
}
```
上のコードはこういうこと
- この State サブクラスに関連付けられたウィジェットが更新されるたびに、もし listPreferences も変更されたら、 _pagingController を refresh() でリフレッシュする

ミッションは達成されました。最後に readwenderlich をビルドして実行し、無限スクロールのページネーションに驚かされるときが来ました。やったね!
![](../../images/readwenderlich-final.gif)

# ここから先は？
チュートリアルの上部または下部にある「資料のダウンロード」ボタンをクリックして、完成したプロジェクトファイルをダウンロードしてください。

さて、ここからが本題です。Infinite Scroll Paginationパッケージには、真ん中のパーツを交換するためのパーツが付属しています。次のレイアウトに最も適したものをお選びください。
![](../../images/additional_pieces.png)
Infinite Scroll Pagination パッケージのすべてのクラスは、ジグソーパズルのピースとして表現されています。

これらの追加ピースに関する詳細は、パッケージのクックブックに記載されています。

ここで提案です。記事を取得するために使用したエンドポイントは、検索クエリを受け付けます。それを使ってreadwenderlichに検索サポートを追加するのはどうでしょうか？

このチュートリアルを楽しんでいただけたら幸いです。もし質問やコメントがあれば、以下のフォーラムでの議論に参加してください。













