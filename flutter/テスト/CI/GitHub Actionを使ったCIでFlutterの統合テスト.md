# How to add Flutter integration test in a CI with GitHub Action ?
DEC 29, 2021
>https://www.etiennetheodore.com/integration-testing-with-ci/
## Introduction
プロジェクトの開発中には、統合テストを行うものが必要です。そしてさらに重要なのは、CIを使用してテストを自動的に実行することです。この記事では、最初の統合テストを作成する方法と、Github Actionを使用してワークフローでテストを実行する方法を説明します。

### Tools we will need:
- Flutter
- An account GitHub

## Simple integration test
CIワークフローを深く掘り下げる前に、簡単な統合テストが必要です。そのために、テストを内包した簡単なアプリケーションを作成します。
```command
flutter create integration_testing_with_ci
```
次に、テストを作成します。
```command line
cd integration_testing_with_ci
mkdir integration_test
mkdir test_driver
```
まず、test_driver/の中にintegration_test.dartを作成します。
- integration_test.dart
```dart
import 'dart:io';
import 'package:integration_test/integration_test_driver_extended.dart';

Future<void> main() async {
  try {
    await integrationDriver(
      onScreenshot: (String screenshotName, List<int> screenshotBytes) async {
        final File image = await File('screenshots/$screenshotName.png').create(recursive: true);
        image.writeAsBytesSync(screenshotBytes);
        return true;
      },
    );
  } catch (e) {
    print('Error occured: $e');
  }
}
```

ここでは、スクリーンショットフォルダを処理するためのコードを追加していることがわかります。

次に、テストそのものである increment_success_test.dart が必要です。
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';

import 'package:integration_testing_with_ci/main.dart' as app;

void main() {
  final binding = IntegrationTestWidgetsFlutterBinding();
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('increment', () {
    testWidgets('success', (WidgetTester tester) async {
      app.main();
      await binding.convertFlutterSurfaceToImage();
      await tester.pumpAndSettle();

      await binding.takeScreenshot('test-screenshot');

      // Verify the counter starts at 0.
      expect(find.text('0'), findsOneWidget);

      // Finds the floating action button to tap on.
      final Finder fab = find.byTooltip('Increment');

      // Emulate a tap on the floating action button.
      await tester.tap(fab);

      // Trigger a frame.
      await tester.pumpAndSettle();

      // Verify the counter increments by 1.
      expect(find.text('1'), findsOneWidget);
    });
  });
}
```
increment_success_test.dart
これは、公式ドキュメントに掲載されている簡単なテストです。

これで、ローカルでこの統合テストをコマンドで実行できるようになりました。
```command
flutter drive --driver=test_driver/integration_test.dart --target=integration_test/increment_success_test.dart
```
を実行
screenshots/ フォルダにスクリーンショットが表示されるはずです。

## Workflow Github Action
今回はGitHubを使用してワークフローを作成します。まず、git init でリポジトリを init し、.github/workflows/ フォルダを test-integration.yaml というファイル名で作成します。

一番重要なアクションはreactivecircus/android-emulator-runner@v2で、ドキュメントは[こちら](https://github.com/ReactiveCircus/android-emulator-runner)にあります。これはアンドロイドのエミュレータを起動し、その上でスクリプトを実行します。

ワークフローを作成します。
```yaml
on:
  push:
    tags-ignore:
      - '**'
    branches:
      - '**'
name: Flutter integration test
jobs:
  drive_android:
    runs-on: macos-latest
    strategy:
      matrix:
        api-level: [29]
        target: [playstore]
    steps:
      - uses: actions/checkout@v2
      - uses: subosito/flutter-action@v1
        with:
          flutter-version: '2.8.1'
          channel: 'stable'

      # Run integration test
      - name: Run Flutter Driver tests
        uses: reactivecircus/android-emulator-runner@v2
        with:
          target: ${{ matrix.target }}
          api-level: ${{ matrix.api-level }}
          arch: x86_64
          profile: Nexus 6
          script: flutter drive --driver=test_driver/integration_test.dart --target=integration_test/increment_success_test.dart

      # Update screenshot to artifact
      - name: Upload screenshots
        if: always()
        uses: actions/upload-artifact@v1
        with:
          name: screenshot
          path: ${{ matrix.path }}screenshots/
```
test-integration.yaml
The most important in this workflow is the job "Run Flutter Driver tests", the action will start an emulator for us then execute the script provided and here we use the same command as before in local.

And finally, we upload the screenshots in artifact so we can check it later.

We just have to push the file into the repository and the workflow will be triggered:















