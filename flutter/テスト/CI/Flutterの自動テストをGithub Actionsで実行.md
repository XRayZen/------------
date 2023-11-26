
>https://goyoki.hatenablog.com/entry/2021/01/23/011844

Flutterのユニットテスト、ウィジェットテスト、インテグレーションテスト(エミュレータを使ったテスト)をGithub Actionsで実行する方法について

## 対象のディレクトリ構成
- (rootディレクトリ)
  - test
    - ユニットテスト、ウィジェットテストのテストコード
  - test_driver
    - app.dart
    - インテグレーションテストのテストコード
- .github
  - workflows
    - flutter_test.yaml
    - flutter_integration_test.yaml
- 他Flutterアプリのプロジェクトファイル

## ユニットテスト、ウィジェットテストのワークフロー

上記ディレクトリファイルにて、testディレクトリのユニットテスト、ウィジェットテストをGithub Actionsで実行する場合、次のようなワークフロー定義ファイル`flutter_test.yaml`を作成します。
push、プルリクをトリガに、単にflutter実行用のアクション上で、テスト実行コマンドを実行しています。
```yaml
name: flutter component test

on: [push, pull_request]

jobs:
  component_test:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2
      - uses: subosito/flutter-action@v1
        with:
          channel: 'stable'
      - run: flutter pub get
      - run: flutter test
```
## インテグレーションテストのワークフロー
次にtest_driverディレクトリのインテグレーションテストをGithub Actionsで実行する場合、次のようなワークフロー定義ファイルflutter_integration_test.yamlを作成します。
この例では「iPhone 12 Pro Max (14.2)」「iPhone 12 (14.2)」のシミュレータでテストを実行します。処理としては、xcrunで利用可能なシミュレータ一覧を取得し、awkでそこから対象のUDIDを取得してエミュレータを起動します。そしてflutter driveコマンドでテスト対象・テストコードを実行します。
```yaml
name: flutter integration test

on: [push, pull_request]

jobs:
  integration_test:
    strategy:
      matrix:
        device:
          - "iPhone 12 Pro Max (14.2)"
          - "iPhone 12 (14.2)"
      fail-fast: false
    runs-on: macos-latest
    steps:
      - name: "start simulator"
        timeout-minutes: 10
        run: |
          UDID=$(
            xcrun instruments -s |
            awk \
              -F ' *[][]' \
              -v 'device=${{ matrix.device }}' \
              '$1 == device { print $2 }'
          )
          xcrun simctl boot "${UDID:?No Simulator with this name found}"
      - uses: actions/checkout@v2
      - uses: subosito/flutter-action@v1
        with:
          channel: 'stable'
      - run: "flutter drive --target=test_driver/app.dart"
```


