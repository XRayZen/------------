
>https://caddi.tech/archives/1373
derive マクロによって、記述量を削減することができました。
具体的には、PartialEq , Eq を derive で自動実装し、 getter を Getters という derive マクロで自動実装しました。

derive マクロは、 std crate 以外にも外部 crate で定義された便利なものがたくさんあります。以下に一部をご紹介します。

derive-getters
フィールドのgetterを自動実装する derive マクロ
公開したくないフィールドは skip attribute で飛ばせる
derive-new
コンストラクタを自動実装する derive マクロ
strum
enumに対する Display, FromStr を自動実装する derive マクロ
derive マクロは便利ですが、時と場合によって使い分ける必要があります。
例えば、先程紹介したderive-new。このマクロは、コンストラクタを自動生成してくれる点で便利ですが、コンストラクタ内にvalidationをはさみたい時は自分で実装する必要があります。