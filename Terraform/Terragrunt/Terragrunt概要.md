# Terraform だけだと大変なので Terragrunt を使おう

## 参考
>[TG公式文法](https://terragrunt.gruntwork.io/docs/reference/config-blocks-and-attributes/)

>https://qiita.com/ssc-ksaitou/items/c3eedd46a5eb04d731cc

>[Terragruntでより幸せなTerraform生活を目指す](https://zenn.dev/nameless_gyoza/articles/terragrunt-hands-on)

>[TerragruntはTerraformのState格納用S3バケットを自動作成してくれる](https://dev.classmethod.jp/articles/terrugrunt-creates-s3-bucket-for-buckend-automatically/)

>[Terragrunt 導入で Terraform コードを DRY](https://zenn.dev/simpleform/articles/20221111-01-terraform-with-terragrunt)

>[CloudBuild で最強のTerraform & Terragrunt CI/CD環境を作る](https://mixi-developers.mixi.co.jp/strongest-terraform-terragrunt-ci-e4c350d627e6)

- [Terraform だけだと大変なので Terragrunt を使おう](#terraform-だけだと大変なので-terragrunt-を使おう)
  - [参考](#参考)
  - [概要](#概要)
  - [Terragruntのインストールと使い方](#terragruntのインストールと使い方)
  - [使い方/適用](#使い方適用)
  - [コマンド](#コマンド)
    - [基本的なコマンド](#基本的なコマンド)
  - [Terragruntプロジェクトの基本的なディレクトリ構造](#terragruntプロジェクトの基本的なディレクトリ構造)
    - [モジュールの定義 (`modules/*/*.tf`)](#モジュールの定義-modulestf)
    - [terragrunt.hcl](#terragrunthcl)
      - [親 terragrunt.hcl](#親-terragrunthcl)
      - [子 terragrunt.hcl](#子-terragrunthcl)
    - [環境ごとの変数定義 (`envs/*/env.hcl`)](#環境ごとの変数定義-envsenvhcl)
  - [環境ごとにモジュールのバージョンを固定する](#環境ごとにモジュールのバージョンを固定する)


## 概要
- Terraform はそのままだと管理が大変
- Terragruntは Gruntwork社 がメンテしているTerraformのラッパーで、Terraformにちょっと足りないマクロ的な機能を追加してくれます。
- Terraformの設定定義をDRYにする
  - providerやbackend関連の定義は ./envs/dev/iam , ./envs/dev/elb などでそれぞれ定義する必要がありますが、それらの定義を1箇所に集約することができる
  - Terraform実行時に付与する引数もDRYにすることができる
- tfstateの異なるmodule間の実行依存を解決する
  - ./envs/dev/iam , ./envs/dev/elb などには本来依存関係があり（例：IAMリソースを作成してからELBリソースを作成する必要がある）、素のTerraformでは実行順序を工夫する必要があります
  - Terragruntが提供する機能を使うことで依存関係を解決した上でplan/applyなどを実行できるようになります
- 各ステートの依存管理が強力です。
  - 通常、DIR_Aで設定した内容をDIR_Bで参照したい場合は自分で順番を制御する必要がありますが Terragrunt は勝手に行ってくれます。
- terraform コマンドのかわりに terragrunt コマンドを使うようにするだけです。
  - 内部的にはTerragruntがお膳立てをした後、最終的にTerraformが呼び出されるだけです。
  - 基本的にはTerraformの使い方を知っていればTerragrunt自体は難しいことはありません。
## Terragruntのインストールと使い方
- Mac
  - `brew install terragrunt`
- Windows
  - `scoop install terragrunt`
## 使い方/適用
- terraform init はTerragruntが勝手にやってくれるので不
- Terragruntは勝手に各モジュールの依存順を計算してくれるので 、いちいち環境ごとにモジュールフォルダを順番に開いて apply せずとも、
  - 下記のように 環境ごとのディレクトリで `terragrunt run-all apply` すれば適用順序を勝手に計算して全て init & apply してくれます
  - 非常に楽
    ```bash
    $ cd envs/stg
    $ terragrunt run-all apply
    The stack at /Users/ksaitou/foobar/envs/stg will be processed in the following order for command apply:
    Group 1
    - Module /Users/ksaitou/foobar/envs/stg/modB/

    Group 2
    - Module /Users/ksaitou/foobar/envs/stg/modA/
    - Module /Users/ksaitou/foobar/envs/stg/modC/

    Are you sure you want to run 'terragrunt apply' in each folder of the stack described above? (y/n)
    ```
  - そのまま y とタイプすれば順番に全モジュールについて terraform init & terraform applyしてくれる
    - また、terragrunt apply はデフォルトで plan オプションが有効になっているので、実際に適用する前に plan が実行されます。
## コマンド
- 複数モジュールへの plan や apply を単一のコマンドで実行するには、run-all コマンドをapplyの前につけるだけです。
  - `terragrunt run-all apply`

!!! attention plan-all や apply-all などのコマンドは現在は非推奨になっており、run-all コマンドの利用が推奨されています。 
### 基本的なコマンド
```bash
% terragrunt init           # 初期化
% terragrunt validate       # 検証
% terragrunt fmt            # コード整形
% terragrunt plan           # Dry-Run
% terragrunt apply          # デプロイ
% terragrunt destroy        # 削除
terragrunt graph-dependencies # 依存関係の有向グラフを確認できます。
```
## Terragruntプロジェクトの基本的なディレクトリ構造
- modules/ の下にモジュールごと(e.g., vpc, db など)ディレクトリを掘る
  - モジュール作り方は通常のterraformモジュールと同じ
- envs/ の下に環境ごと(e.g., stg, prod など)ディレクトリを掘る
  - 直下に環境全体の変数定義を収める env.hcl を置く
  - 参照するモジュールのディレクトリをそれぞれ下に掘って、その中にモジュールの参照定義となる terragrunt.hcl を作成する
```tree
modules/
  modA/
    *.tf
  modB/
  modC/
envs/
  terragrunt.hcl //共通的な設定（バックエンド情報など）を親ファイルに記述
  stg/
    env.hcl
    modA/
      terragrunt.hcl # 子
    modB/
    modC/
  prod/
    modA/
    modB/
    modC/
```
- 親ファイルの場所を envs/ 配下ではなく、あえてその上に置いています。
  - 意図としては、modules 配下の各モジュールにおいて実装サンプルの Dry-Run (plan) を実行する場合にも、同じ親ファイルにアクセスできるようにするためです。
### モジュールの定義 (`modules/*/*.tf`)
通常のTerraformのモジュール定義と同じ
下記のようにファイルを切ることが多い
- `input.tf`: 
  - 入力パラメータ
- `output.tf`: 
  - 出力値
- `機能名.リソース種別.tf`: 
  - 例:`featureA.route53.tf`
>`input.tf`
```bash
variable "instance_count" {
description = "How many servers to run"
}

variable "instance_type" {
description = "What kind of servers to run (e.g. t2.large)"
}
```
>`機能名.リソース種別.tf`
```bash
provider "aws" {
region = "ap-northeast-1"
}

resource "aws_instance" "web" {
ami = "ami-afb09dc8"
instance_type = "${var.instance_type}"
count = "${var.instance_count}"
}
```
>`output.tf`
```bash
output "instance_ip_addr" {
value = "${aws_instance.web.*.public_ip}"
}
```
このモジュールのディレクトリはそれぞれ `envs/*/*/terragrunt.hcl` （後述） から参照します。
### terragrunt.hcl
- Terragrunt でリソース管理を行うには、バックエンド設定などを記述する terragrunt.hcl というファイルが必要になります。
  - terragrunt.hcl には 1 つの親ファイルと複数の子ファイルがあり、共通的な設定（バックエンド情報など）を親ファイルに記述します。
    - これを複数の子ファイルから参照することで DRY な記述を実現しています。
#### 親 terragrunt.hcl
- 全環境共通の定義 (`envs/terragrunt.hcl`)
- 親ファイルでまず重要なのはバックエンドの記述です。
  - Terragrunt では terragrunt.hcl の [remote_state](https://terragrunt.gruntwork.io/docs/reference/config-blocks-and-attributes/#remote_state) ブロックに記述します。
- 特徴的なのは tfstate のキー指定です。
  - バックエンド設定の中で唯一共通化できない部分ですが、[path_relative_to_include 関数](https://terragrunt.gruntwork.io/docs/reference/built-in-functions/#path_relative_to_include)によって親ファイルから include ブロックを持つ子ファイルまでの相対パスを取得し、これをキーに含めることで共通化を実現しています。
- 下記のような環境全体の定義ファイルを置きます。
  - 実際に各環境ごとのモジュールを適用する際に実行されます。
  - やっていることは簡単で、各環境のプロバイダやTerraform自体の設定のファイル 1 のメタテンプレート定義をしているだけです。
>envs/terragrunt.hcl - 親
```bash
# 各環境ごとの *.tfstate の入れ方についての定義
remote_state {
  backend = "s3"

  config = {
    region = "ap-northeast-1"
    bucket = "foobar-tfstate"
    # 下記のようにしておくと、例えば
    # `envs/stg/modA` の tfstate は
    # `foobar/stg/modA.tfstate` に格納されるようになる
    key    = "foobar/${path_relative_to_include()}.tfstate"
    # Credentialを別ファイルに分けて管理する場合は下記のようにする
    profile = "foobar-terraform"
  }

  generate = {
    path      = "_backend.tf"
    if_exists = "overwrite"
  }
}
# Terraform リソースが利用する provider 設定の記述
# contents にヒアドキュメントとして記述した内容を path に指定したファイル名でファイル生成します。
# ここでは AWS の provider を利用することを想定している
generate "provider" {
  path      = "_provider.tf"
  if_exists = "overwrite_terragrunt"

  contents = <<EOF
terraform {
  required_version = ">= 1.3.7"

  required_providers {
    aws = {
      # See https://github.com/terraform-providers/terraform-provider-aws
      version = "~> 4.50.0"
    }
  }
}

provider "aws" {
  profile = "foobar-terraform"
  region  = "ap-northeast-1"

  default_tags {
    tags = {
      Project = "foobar"
    }
  }
}
EOF
}
```
!!! note: contents にはヒアドキュメントではなく、ファイルパスを指定することもできます。上記の provider 設定では、同ファイル内の locals ブロックに定義した変数にアクセスするため、ヒアドキュメントとしています。
!!! note: 生成されるファイル名を _provider.tf のように先頭に _ をつけているのは、.gitignore に _*.tf を追加して Git 管理対象外としたかったという意図です。Terragrunt によって自動生成されるファイルは基本的に編集する必要がなく、誤編集されるリスクを軽減するためです。
#### 子 terragrunt.hcl
- 環境とモジュールごとの定義 (`envs/*/*/terragrunt.hcl`)
- 各環境ごとに利用するモジュールの呼び出し定義を配置します。
>envs/stg/modA/terragrunt.hcl
```bash
# 子ファイルで重要な記述
# 全環境の定義 (`envs/terragrunt.hcl`) をインクルードする
# これによって後述の親ファイルの内容を継承でき、子ファイル側でバックエンド設定の記述を省略することができます。
include "root" {
  # find_in_parent_folders 関数は、カレントディレクトリから親ディレクトリを再起的に探索し、最初に見つけた terragrunt.hcl の絶対パスを返します。
  # また、関数の引数に任意のファイル名を指定することができ、その場合は terragrunt.hcl の代わりに指定したファイル名で同様の挙動を示します。
  # 該当するファイルが親ディレクトリを辿っても存在しない場合はエラーになります。
  path = find_in_parent_folders()
}

# 環境の定義 (`env.hcl`) を local.env.locals として参照できるようにする
locals {
  # HCL ファイルに設定を記述し、以下のように read_terragrunt_config 関数で読み込むこともできます
  env = read_terragrunt_config(find_in_parent_folders("env.hcl"))
}

# モジュールを参照する
# モジュール呼び出しの際は、terraform ブロックの source にモジュールへのパスを指定
terraform {

  # source = "${dirname(find_in_parent_folders())}//modules/s3_bucket"
  # ソース:https://zenn.dev/simpleform/articles/20221111-01-terraform-with-terragrunt#%E6%83%B3%E5%AE%9A%E3%81%99%E3%82%8B%E6%A7%8B%E6%88%90
  source = "../../../modules//modA"
}
# 依存するリソースは dependency ブロックで定義
# 他のモジュールの出力値を参照する
# (素のTerraformだと面倒な作業の一つ)
dependency "modB" {
  config_path = "${dirname(find_in_parent_folders())}/envs/dev/s3_bucket"
  config_path = "../modB"
  # 依存する s3_bucket が Apply される前に s3_bucket_acl を Plan する場合、
  # outputs の値が確定しておらず、エラーになります。
  # エラーを回避するには、dependency ブロックの中に mock_outputs を定義します。
  # plan時などには`../iam_role`から値が分かってこないのでmockを利用しておくことで型としての埋め合わせができます
  mock_outputs = {
     bucket_name = "hoge"
  }
}

# モジュールの入力値を指定する
# モジュールの variables.tf で定義された入力は inputs として渡します
# モジュールの出力値がデプロイされないと分からない場合は依存を定義する必要があります
inputs = {
  bucket_name = "${local.org}-example-bucket-${local.env}"
  env                   = local.env.locals.env
  # 依存するモジュールの出力には dependency.<NAME>.outputs.<OUTPUT_NAME> のようにアクセスできます   
  backup_bucket         = dependency.modB.outputs.backup_bucket
  backup_retention_days = local.env.locals.backup_retention_days
}
``` 
### 環境ごとの変数定義 (`envs/*/env.hcl`)
>envs/stg/env.hcl
```bash
# 検証環境
locals {
  # 環境名
  env = "stg"
  # バックアップ保持日数
  backup_retention_days = 3660
}
```
## 環境ごとにモジュールのバージョンを固定する
下記のように検証環境のみTerraformモジュールの最新版で開発し、本番環境はリリースタイミングまでは古いバージョンで固定したいケースがあると思いますが、そういったニーズもTerragruntは実現可能です。
- stg
  - 常に main ブランチの内容を適用したい
- prod
  - 特定のタグの内容を適用したい
- dev
    - 特定のコミットの内容を適用したい

環境ごとのモジュールのファイル terragrunt.hcl の terraform { ... } のモジュールのパス参照部分を下記のように書き換えるだけです。

>envs/prod/modA/terragrunt.hcl
```bash
terraform {
  // 直接ディレクトリを参照する場合
  // source = "../../../modules//modB"

  // 特定のgitリポジトリを参照する場合 (リモートリポジトリの参照のみ可能)
  // source = "git::gitリポジトリURL//リポジトリ内のディレクトリパス?ref=コミットの参照"
  source = "git::ssh://git@github.com:ssc-ksaitou/foobar-modules.git//modules/modB?ref=1.0.0"
}
```

