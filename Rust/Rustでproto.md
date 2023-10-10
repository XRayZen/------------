# Rustでprotobuf
>https://www.getto.systems/entry/2021/04/26/170845
## 1. `Cargo.toml`
1. `[package]`セクションに`build = "build.rs"`を追加
    ```toml
    [package]
    name = "hello"
    version = "0.1.0"
    edition = "2021"
    build = "build.rs"
    ```
2. proto関連を追加
    ```toml
    [dependencies]
    bytes = "1.4.0"
    prost = "0.11.9"
    protobuf = "3.2.0"
    [build_dependencies]
    prost-build = { version = "0.11.9"}
    protobuf-codegen = "3.2.0"
    ```
## 2. `build.rs`を作成
- `build.rs`は`cargo build`時に実行される
- プロジェクトのルートに`build.rs`を作成
```rust
use std::{io::Result, fs};

fn main() -> Result<()> {
    println!("build proto code");
    // 生成先のディレクトリを作成
    let output_dir = "src/gen_proto";
    // protoファイルが入ってるディレクトリを指定
    let proto_input_dir = "proto";
    fs::create_dir_all(output_dir)?;
    protobuf_codegen::Codegen::new()
        .out_dir(output_dir)
        .include(proto_input_dir)
        // 生成するprotoファイルを指定
        .input("proto/hello_req.proto")
        .input("proto/hello_res.proto")
        .run()
        .expect("Unable to generate hello.rs");
    Ok(())
}
```
## 3. `proto`ディレクトリを作成
- `build.rs`で指定した`proto/hello.proto`を作成
```protobuf
syntax = "proto3";

package hello;

message HelloRequest {
    string name = 1;
}

message HelloResponse {
    string message = 1;
}

service HelloService {
    rpc Hello(HelloRequest) returns (HelloResponse);
}
```
## 4. `src/gen_proto`に`hello.rs`が生成される
- `src/gen_proto/hello.rs`が生成される

## 5. 受信したprotoをパースする
- `src/main.rs`に以下を追加
```rust
    // このデータをprotobufでデコードする
    let request: HelloRequest = protobuf::Message::parse_from_bytes(data)?;
```
## 6. protoを返す
```rust
async fn get_user_process<'a, T: RepoTrait>(
    request: HelloRequest,
    usecase: Usecase<'a, T>,
) -> Result<Response<Body>, Box<dyn std::error::Error + Send + Sync + 'static>> {
    let mut hello_res = gen_proto::hello_res::HelloResponse::new();
    let mut get_user_response = gen_proto::hello_res::GetUserResponse::new();
    get_user_response.message = usecase
        .get(&request.get_user_name_request().user_id)
        .await?;
    hello_res.set_get_user_response(get_user_response);
    hello_res.api_res_type =
        EnumOrUnknown::new(gen_proto::hello_res::ApiResType::API_RES_TYPE_SUCCESS);
    let response_data = hello_res.write_to_bytes()?;
    return Ok(Response::builder()
        .status(200)
        .header("content-type", "application/x-protobuf")
        .body(response_data.into())
        .map_err(Box::new)?);
}
```












