- [はじめてのChakra UI](#はじめてのchakra-ui)
- [プロバイダのセットアップ](#プロバイダのセットアップ)

# はじめてのChakra UI
>https://reffect.co.jp/react/chakra-ui/

Chakra UI は UI ライブラリの一つでコンポーネントを利用して効率よくユーザインターフェイスを構築することができます。

Chakra UI には 1 つのコンポーネントから作成できる Button コンポーネントから複数のコンポーネントを組み合わせて作成できる Modal コンポーネントまで用途毎にさまざまなコンポーネントが提供されています。
- どのようなコンポーネントが提供されているのかはドキュメントの [Components ページから一覧](https://chakra-ui.com/docs/components)で確認することができます。
>以下コマンドをプロジェクトフォルダで実行
```bash
npm i @chakra-ui/react @chakra-ui/next-js @emotion/react @emotion/styled framer-motion
```
# プロバイダのセットアップ
Next.js 13では、新しいアプリ／ディレクトリ／フォルダ構造が導入されました。
- デフォルトでは、サーバーコンポーネントが使用されます。

!!! info ただし、Chakra UIはクライアントサイドコンポーネントでのみ動作します。

サーバーコンポーネントでChakra UIを使用するには、ファイルの先頭に`'use client';`を追加して、クライアントサイドコンポーネントに変換する必要があります。

また、@chakra-ui/next-jsパッケージは、appディレクトリでChakra UIを使用する際に、よりスムーズなエクスペリエンスを提供します。
- このパッケージは、Emotion.js のキャッシュプロバイダーと next/navigation の useServerInsertedHTML フックを合成する CacheProvider を提供します。
  - これは、計算されたスタイルが最初のサーバーペイロード (ストリーミング中) に含まれるようにするために必要です。

アプリディレクトリで Chakra UI を使用するには、ChakraProvider と CacheProvider を独自のクライアント Component でラップします。
>app/providers.tsx
```typescript
'use client';

import { CacheProvider } from '@chakra-ui/next-js'
import { ChakraProvider } from '@chakra-ui/react'

export function Providers({ 
    children 
  }: { 
  children: React.ReactNode 
  }) {
  return (
    <CacheProvider>
      <ChakraProvider>
        {children}
      </ChakraProvider>
    </CacheProvider>
  )
}
```
これで、ルート内で `<Providers />` を直接インポートしてレンダリングすることができます。
- ルートでレンダリングされたプロバイダにより、Chakra UIのすべてのコンポーネントとフックは、クライアント・コンポーネント内で期待どおりに動作します。
```typescript
// app/layout.tsx
import { Providers } from "./providers";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <Providers>
          {children}
        </Providers>
      </body>
    </html>
  );
}
```
!!! warning use client'ディレクティブは、ページ関連ファイルの先頭に追加する必要がある。
    新しいapp/ディレクトリを使用しない場合は、ファイルの先頭に'use client';ディレクティブを追加する必要はありません。


