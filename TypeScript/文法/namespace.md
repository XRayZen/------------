# namespace（内部モジュール）
- JavaScriptでは名前空間を区切るには関数を使ったトリックが必要でした。
- ですが、TypeScriptではそのトリックを自動で行ってくれる仕組みがあります。
- それが「namespace（内部モジュール）」です。
- 今後は、なるべくnamespaceを使っていくのがよい
- 内部モジュールと対をなす「外部モジュール」という仕組みもありますが（後述）、Webアプリの開発を行う場合、主に使うのはnamespace（内部モジュール）だけでよいでしょう*2。
- *最近はBrowserifyなどの台頭により、外部モジュールを使ってコードを書き、Browserifyで処理して使うワークフローも考えられます。
```typescript
namespace sampleA {
  export var str = "string";
}

window.alert(sampleA.str);
// window.alert(str); // SampleAの中で定義したものは他の場所では参照できない

namespace sampleB {
  export class Hoge {
    hello(word: string): string {
      return "Hello, " + word;
    }
  }
  class Fuga { }
  export interface IMiyo {
    hello(word: string): string;
  }
}

namespace sampleC {
  // SampleB.Hoge を Piyo としてインポート
  import Piyo = sampleB.Hoge;
  import Fuga = sampleB.Fuga; // exportしていないものは参照できない
  import Miyo = sampleB.IMiyo; // インタフェースもimportできる

  export var str = new Piyo().hello("TypeScript");
}

window.alert(sampleC.str);

// TypeScript 1.5.0-beta までは以下の書き方だった。今も使えるが、なるべくnamespaceを使う。
module sampleD {
}
```