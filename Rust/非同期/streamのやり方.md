# async-streamクレートを使用

- 非同期fnからimpl Streamを作成することも可能です。

>sync::mpsc::Receiver を impl Stream に変換します。

```rust
use tokio::sync::mpsc;

let (tx, mut rx) = mpsc::channel::<usize>(16);

let stream = async_stream::stream! {
    while let Some(item) = rx.recv().await {
        yield item;
    }
};
```

## 要素の非同期ストリーム

- stream!とtry_stream!という2つのマクロを提供し、呼び出し側が要素の非同期ストリームを定義できるようにします。
- これらは、async & await記法を用いて実装されています。
- このクレートは、不安定な機能なしで動作します。
- stream! マクロは Stream 特性を実装した無名型を返します。
- Item 関連型は、ストリームから得られる値の型である。
- try_stream! マクロも Stream 特性を実装した無名型を返しますが、Item の型は Result<T, Error> となります。
- try_stream! マクロは、実装の一部として ?表記の使用をサポートしています。

### 使用方法

- 数値を返す基本的なストリーム
- 値はyieldキーワードを使用して生成される。
- ストリームブロックは()を返す

```rust
use async_stream::stream;

use futures_util::pin_mut;
use futures_util::stream::StreamExt;

#[tokio::main]
async fn main() {
    let s = stream! {
        for i in 0..3 {
            yield i;
        }
    };
    //ストリームはpinしなければいけない
    pin_mut!(s); // needed for iteration

    while let Some(value) = s.next().await {
        println!("got {}", value);
    }
}
```

### リターンがストリームの関数

- `impl Stream<Item = T>`を使用してストリームを返すことができる。

```rust
fn zero_to_three() -> impl Stream<Item = u32> {
    stream! {
        for i in 0..3 {
            yield i;
        }
    }
}

#[tokio::main]
async fn main() {
    let s = zero_to_three();
    pin_mut!(s); // iterに必要

    while let Some(value) = s.next().await {
        println!("got {}", value);
    }
}
```

- ストリームは、他のストリームを利用して実装することもできます。

```rust
fn zero_to_three() -> impl Stream<Item = u32> {
    stream! {
        for i in 0..3 {
            yield i;
        }
    }
}

fn double<S: Stream<Item = u32>>(input: S)
    -> impl Stream<Item = u32>
{
    stream! {
        for await value in input {
            yield value * 2;
        }
    }
}

#[tokio::main]
async fn main() {
    let s = double(zero_to_three());
    pin_mut!(s); // needed for iteration

    while let Some(value) = s.next().await {
        println!("got {}", value);
    }
}
```

- Rustのtry記法(?)はtry_stream! マクロで使うことができる。
- 返されるストリームのItemはResultで、Okが返される値、Errは?が返すエラータイプです

```rust
fn bind_and_accept(addr: SocketAddr)
    -> impl Stream<Item = io::Result<TcpStream>>
{
    try_stream! {
        let mut listener = TcpListener::bind(addr).await?;

        loop {
            let (stream, addr) = listener.accept().await?;
            println!("received on {:?}", addr);
            yield stream;
        }
    }
}
```

## 実装方法

- stream!マクロとtry_stream!マクロは、procマクロを使用して実装されています。
- このマクロは、```sender.send($expr)``` のインスタンスを構文木から検索し、 ```sender.send($expr).await``` に変換しています。

- ストリームは、軽量な送信者を使用して、ストリーム実装から呼び出し元へ値を送信します。
- ストリームに入るとき、```Option<T>```がスタックに格納されます。
- セルへのポインタはスレッドローカルに格納され、pollは非同期ブロック上で呼び出されます。
- pollが戻ると、```sender.send(value)```はそのcellの値を格納し、呼び出し元に戻すことをyieldします。
