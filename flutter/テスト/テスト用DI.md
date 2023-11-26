# 通常の依存性注入とテスト時の依存性上書き
1. 通常時はProvider内で普通にインフラ層実装RepoをビジネスロジックにDIしてインスタンス化すれば良い
    ```dart
    final contentImageLogicProvider = Provider.autoDispose((ref) {
        return ContentImageLogic(
        apiRepository: ref.read(implBackEndApiRepoProvider),
        ioRepository: ref.read(implFileIORepoProvider),
        config: ref.read(appConfigProvider));
        });
    ```
2. テスト時は`Provider.Scope`の`overrides`パラメータを使って、`XXXrepositoryProvider`の挙動をモックでオーバーライドする
    ```dart
    final mockContentImageLogicProvider = ContentImageLogic(
        apiRepository: MockBackEndApiRepoProvider,
        ioRepository: MockFileIORepoProvider,
        config: MockAppConfigProvider
    );
    //tester.pumpWidgetにラップして書く必要がある
    await tester.pumpWidget(ProviderScope(
      overrides: [
        contentImageLogicProvider
            .overrideWithValue(mockContentImageLogicProvider)
      ],
      //テストしたいWidgetを選択
      child: testwidget(),
    ));
    ```