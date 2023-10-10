# AWS Lambda における Go の init() の振る舞い
>https://michimani.net/post/aws-behavior-of-go-init-func-on-lambda/

# まとめ
AWS Lambda のウォームスタート時には Go の init() 関数は実行されません。
実行ごとに初期化したい変数がある場合は、グローバル変数で定義せずにハンドラ内で宣言・初期化する、もしくは beforeHandler() みたいな初期化用の関数を作って、ハンドラの最初でそれを実行するみたいなことをする必要がありそうです。
