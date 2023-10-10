# Streams

><https://zenn.dev/magurotuna/books/tokio-tutorial-ja/viewer/streams>
><https://blog.ymgyt.io/entry/mini_redis_tutorial_to_get_started_with_tokio>

- "stream" は非同期な値の連なりです。
- Rust の std::iter::Iterator の非同期バージョンに相当するもの
- Stream トレイトによって表現されます。
- "stream" は async 関数の中でイテレートすることができます。
- また、アダプタを利用して変形させることも可能です。
- Tokio は StreamExt トレイトを通して多くの共通アダプタを提供しています。

Tokio の "stream" サポートは別のクレート tokio-stream によって提供されています。

```yaml
tokio-stream = "0.1"
```

>!現在、Tokio の "stream" ユーティリティは tokio-stream クレート内にあります。  
Rust の標準ライブラリの Stream トレイトが安定化したら、Tokio の "stream" ユーティリティは tokio 本体のクレートへと移動される予定です。

## イテレーション

- 現在、Rust は非同期な for ループをサポートしていません。  
- 代わりに、StreamExt::next() と while let を組み合わせて使うことで "stream" に対するイテレーションを行えます。
- イテレータと同様、next() メソッドは Option<T> を返します。
- ここで、T は "stream" の値の型です。
- None が返ってきたら、"stream" のイテレーションが終了したということ

```rust
use tokio_stream::StreamExt;

#[tokio::main]
async fn main() {
    let mut stream = tokio_stream::iter(&[1, 2, 3]);

    while let Some(v) = stream.next().await {
        println!("GOT = {:?}", v);
    }
}
```

## Mini-Redis のブロードキャストとアダプタ

- mini-redis-serverを起動
- アダプタを使えばサンプル2のコードを短く出来る
- Stream を受け取って別の Stream を返す関数のことをしばしば「ストリームアダプタ」と呼びます。
- 「アダプタパターン」[1]の形をしているからです。
- 一般的なストリームアダプタには map、take、filter などがあります。

```rust
let messages = subscriber
    .into_stream()
    .take(3);
```

- メッセージを3つ受け取ったあと、メッセージのイテレートを中断するようにします。
- これは take を使うことで実現できます。
- このアダプタは、最大で n 個のメッセージを送出するように "stream" を制限

サンプル2

```rust
use tokio_stream::StreamExt;
use mini_redis::client;

async fn publish() -> mini_redis::Result<()> {
    let mut client = client::connect("127.0.0.1:6379").await?;

    // いくつかのデータを発行する
    client.publish("numbers", "1".into()).await?;
    client.publish("numbers", "two".into()).await?;
    client.publish("numbers", "3".into()).await?;
    client.publish("numbers", "four".into()).await?;
    client.publish("numbers", "five".into()).await?;
    client.publish("numbers", "6".into()).await?;
    Ok(())
}

async fn subscribe() -> mini_redis::Result<()> {
    let client = client::connect("127.0.0.1:6379").await?;
    let subscriber = client.subscribe(vec!["numbers".to_string()]).await?;
    let messages = subscriber.into_stream();
    //streamをpin!しないとエラーで怒られる
    tokio::pin!(messages);

    while let Some(msg) = messages.next().await {
        println!("got = {:?}", msg);
    }

    Ok(())
}

#[tokio::main]
async fn main() -> mini_redis::Result<()> {
    tokio::spawn(async {
        publish().await
    });

    subscribe().await?;

    println!("DONE");

    Ok(())
}
```

