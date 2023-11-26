# Flutter channel の確認方法と切り替え方法
>https://zuma-lab.com/posts/flutter-channel-command
- channel コマンド で Flutter SDK の 安定版や β 版を切り替えることができます。
- 以下コマンドでも確認できます。
`flutter channel`
## Flutter channel
- master
  - 現在の最新ソースがコミットされる開発ブランチ。最新の機能が取り込まれる反面、もっとも不安定ともいえるチャンネル
- dev
  - master の次フェーズに相当する最新のテスト済みブランチ。一定の品質は保たれているので、master よりは安心して使えるチャンネル
- beta
  - 前月で最も品質の良かった dev チャンネルのビルド。月 1 回のアップデートと dev チャンネルよりも更新は遅いが、品質的にはより安定したチャンネル
- stable
  - 利用が推奨される安定版チャンネル。四半期に 1 回アップデートされる
- 基本は stable を開発することになりますが、stable で取り込まれていない bugfix や新機能がある場合、beta や dev に切り替える場面が出てきます。
- channel の切り替えは以下コマンドを実行します。
  - `flutter channel {channel名}`
- channel を切り替えたら以下コマンドを実行して切り替えた channel の Flutter SDK を最新化します。
  - `flutter upgrade`
- また、channel の切り替えにより package が変更になっている場合は以下コマンドで package も更新します。
  - `flutter pub get`
# Flutter Version 確認方法
`flutter --version`
- 実行結果には 現在使用している chanel や Dart の version も併せて表示されます。
```powershell
Flutter 1.22.5 • channel stable • https://github.com/flutter/flutter.git
Framework • revision 7891006299 (7 weeks ago) • 2020-12-10 11:54:40 -0800
Engine • revision ae90085a84
Tools • Dart 2.10.4
```
