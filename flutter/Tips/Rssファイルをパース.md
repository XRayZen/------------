
>https://future-architect.github.io/articles/20210514a/
1. コマンド実行
`flutter pub add webfeed`
`flutter pub add ogp_data_extract`
1. OGP画像のURLを取得する
```dart
Future<String?> getOGPImageUrl(String url) async {
  final data = await OgpDataExtract.execute(url);
  return data?.image;
}
```
2. パースする
```dart
List<RssFeedItem> convertXmlToRss(ByteData data) {
  final xml = utf8.decode(data.buffer.asUint8List());
  final rss = RssFeed.parse(xml);
  final title = rss.title;
  return [];
}
```
## XmlからRssをパースする
```dart
Future<List<RssFeedItem>> convertXmlToRss(ByteData data) async {
  final xml = utf8.decode(data.buffer.asUint8List());
  final rss = RssFeed.parse(xml);
  final items = <RssFeedItem>[];
  for (var i = 0; i < rss.items!.length; i++) {
    final item = rss.items![i];
    //サムネ画像を取得するためにリンクhtmlを読み込んでjpg画像リンクをパースして
    //それのリンクを入れてUI時にダウンロードして表示する
    final ogpLink = await getOGPImageUrl(item.link!);
    String imageLink;
    if (item.content != null) {
      imageLink = item.content!.images.first;
    } else {
      imageLink = ogpLink ?? '';
    }
    items.add(
      RssFeedItem(
        index: i,
        title: item.title ?? '',
        description: item.description ?? '',
        link: item.link ?? '',
        image: RssFeedImage(link: imageLink, image: ByteData(0)),
        site: rss.title ?? '',
        category: rss.dc?.subject ?? '',
        lastModified:
            item.pubDate ?? item.dc?.date ?? DateTime.utc(2000, 1, 1, 1, 1),
      ),
    );
  }
  return items;
}
```
```dart
Future<String?> getOGPImageUrl(String url) async {
  final data = await OgpDataExtract.execute(url);
  final meta = await MetadataFetch.extract(url);
  if (data == null || meta == null) {
    return parseImageThumbnail(await httpLoadDoc(url));
  }else{
    return data.image ?? meta.image;
  }
}
```
```dart
String parseImageThumbnail(Document doc) {
  // ヘッダー内のtitleタグの中身を取得
  // final title = doc.head!.getElementsByTagName('title')[0].innerHtml;
  String _description;
  var imageLink = '';
// ヘッダー内のmetaタグをすべて取得
  var metas = doc.head!.getElementsByTagName('meta');

  for (var meta in metas) {
    // metaタグの中からname属性がdescriptionであるものを探す
    if (meta.attributes['name'] == 'description') {
      _description = meta.attributes['content'] ?? '';
      // metaタグの中からproperty属性がog:imageであるものを探す
    } else if (meta.attributes['property'] == 'og:image') {
      imageLink = meta.attributes['content']!;
    }
  }
  return imageLink;
}
```


