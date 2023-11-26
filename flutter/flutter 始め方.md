# flutter環境構築
1. ググってやる

# 新たなプロジェクト
1. F1でflutterを入れて新しいプロジェクトを開始する
2. はじめのうちに静的解析強化を入れておく
3. flutter pub add tuple
4. riverpodもインストール

## flutterのprotoc_dartのセットアップ
1. コマンドプロンプトかパワーシェルにコマンドを打ち込む
- `dart pub global activate protoc_plugin`
2. [ここに](https://github.com/protocolbuffers/protobuf/releases "タイトル")をダウンロードする
3. C:\flutterに展開する
4. 環境変数に/flutter/binを追加する
5. proto_cプラグインのインストール場所にprotoc.exeをコピーする
- [参考元](https://gayan-justo.medium.com/working-with-grpc-with-flutter-on-windows-17493528e6a2 "tai")
- コマンド例
- `protoc --dart_out=grpc:dart出力パス -I proto元パス proto名.proto`

