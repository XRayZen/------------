# RustのCargo.tomlの場所がrootに無い時にVSCodeでrust-analyzerのエラーを回避する方法
>[RustのCargo.tomlの場所がrootに無い時にVSCodeでrust-analyzerのエラーを回避する方法](https://zenn.dev/razokulover/scraps/17844b5b5c7147)

通常のrustのプロジェクトだとCargo.tomlはルートディレクトリに配置されていると思うが、monorepoのようになっていた場合にmonorepoのルートでVSCodeを立ち上げるとrust-analyzerがrust-analyzer failed to discover workspaceというエラーを吐く。

これを修正するためにはルートに.vscode/settings.jsonを作成し、下記のような感じでCargo.tomlの場所を絶対指定のパスで直接指定すれば良い。これでエラーは回避できる。
```json
{
    "rust-analyzer.linkedProjects": [
        "./client/api_client/Cargo.toml",
        "./lambda/hello/Cargo.toml",
        "./lambda/world/Cargo.toml",
    ],
}
```
# error: linking with `cc` failed: exit status: 1
>https://zenn.dev/ash/scraps/ced4e3a5c3d392
>https://xn--kst.jp/blog/2021/05/16/m1mac-cross-compile/
クロスコンパイル用のツールをインストール

>https://github.com/messense/homebrew-macos-cross-toolchains

```bash
mkdir -p $HOME/tools && cd $HOME/tools # 適当なディレクトリ
```
## 2. [ツールチェイン](https://github.com/messense/homebrew-macos-cross-toolchains/releases)をダウンロード
# x86_64-unknown-linux-musl-aarch64-darwin
```bash
wget https://github.com/messense/homebrew-macos-cross-toolchains/releases/download/v11.2.0-1/x86_64-unknown-linux-musl-aarch64-darwin.tar.gz
```

## 3. 解凍
```bash
tar xvf x86_64-unknown-linux-musl-aarch64-darwin.tar.gz
```

## 4. パスを通す
```bash
open ~/.zshrc
```
下記を追記
```bash
# Rust cross
export PATH="$HOME/tools/x86_64-unknown-linux-musl/bin:$PATH"
export CC_x86_64_unknown_linux_musl=x86_64-unknown-linux-musl-gcc
export CXX_x86_64_unknown_linux_musl=x86_64-unknown-linux-musl-g++
export AR_x86_64_unknown_linux_musl=x86_64-unknown-linux-musl-ar
export CARGO_TARGET_X86_64_UNKNOWN_LINUX_MUSL_LINKER=x86_64-unknown-linux-musl-gcc
```
パスを適用
```bash
source ~/.zshrc
```
カーゴにもパスを通す
```bash
$ mkdir .cargo
$ echo '[target.x86_64-unknown-linux-musl]
> linker = "x86_64-linux-musl-gcc"' > .cargo/config
```
# Rustのクロスコンパイル
```bash
cargo build --target x86_64-unknown-linux-musl
```