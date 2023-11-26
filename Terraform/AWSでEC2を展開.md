# TerraformでAWSのEC2をプロビジョニング
>https://dev.classmethod.jp/articles/terraform-getting-started-with-aws/
1. AWS CLIをインストールする
   1. https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html
2. AWSの設定を行う
   1. https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-quickstart.html
   2. AWSユーザー
      1. AWS アクセスポータルの URL: https://d-906793e258.awsapps.com/start, 
      2. ユーザー名: April, 
      3. ワンタイムパスワード: 
      4. パス：
3. テンプレートファイルを作成する
    1. 適当なディレクトリを作成して、その中にTerraformのテンプレートファイルを作成します。
    2. テンプレートファイルの名前は任意ですが、拡張子は*.tfとします。(Terraformはこの*.tfファイルを自動的にテンプレートとして認識してくれます)。
    ```bash
    $ mkdir terraform-test
    $ cd terraform-test
    $ touch main.tf
    ```
   - テンプレートはHCL (HashiCorp Configuration Language)というHashiCorp製の独自言語で記述
   - 独自言語とはいえあくまでも設定ファイル記述用のDSLでJSONとも互換性があるので、習得コストは極めて低い
## テンプレートファイルの内容
```terraform
# AWSのリージョンを指定
provider "aws" {
  region = "ap-northeast-1"
}

# EC2インスタンスを作成
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

# VPCを設定

```
- provider
  - Terraformのプロバイダーを指定
  - 今回はAWSのプロバイダーを指定





