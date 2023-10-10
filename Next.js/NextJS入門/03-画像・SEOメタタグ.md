# 画像の表示
画像を表示する場合は Image コンポーネントを利用してブラウザ上に表示する方法を確認します。
本文書ではフリー画像を利用するために unsplash.com を利用して画像をダウンロードして public ディレクトリに保存します。
- 名前は microsoft365.jpg で保存しています。

Image コンポーネントを利用するためには next/image から import する必要があります。
- props の width と height を設定します。
```typescript
import Image from 'next/image';
import Link from 'next/link';
export default function Home() {
  return (
    <div>
      <ul>
        <li>
          <Link href="/about">
            <a>About</a>
          </Link>
        </li>
      </ul>
      <h1>Hello Next.js</h1>
      <Image src="/microsoft365.jpg" width={500} height={300} />
    </div>
  );
}
```
public ディレクトリに保存した画像を表示させることができました。

次は unsplash.com の画像のリンクを設定します。
```typescript
import Image from 'next/image';
import Link from 'next/link';
export default function Home() {
  return (
    <div>
      <ul>
        <li>
          <Link href="/about">
            <a>About</a>
          </Link>
        </li>
      </ul>
      <h1>Hello Next.js</h1>
      <Image
        src="https://images.unsplash.com/photo-1640622842223-e1e39f4bf627?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=MnwxfDB8MXxyYW5kb218MHx8fHx8fHx8MTY0MjY4OTkyMQ&ixlib=rb-1.2.1&q=80&utm_campaign=api-credit&utm_medium=referral&utm_source=unsplash_source&w=1080"
        width={500}
        height={300}
      />
    </div>
  );
}
```
ブラウザで確認すると Server Error が表示されます。
- `”on `next/image`, hostname "images.unsplash.com" is not configured under images in your `next.config.js` See more info: https://nextjs.org/docs/messages/next-image-unconfigured-host”`

next.config.js ファイルは Next.js に関する設定ファイルで環境変数の設定などを行うことができます。
このエラーを解消するためには next.sj の設定ファイルである next.config.js ファイルに画像のリンク先のドメイン名を登録する必要があります。
```typescript
module.exports = {
  images: {
    domains: ['images.unsplash.com'],
  },
};
```
next.config.js ファイルを更新した場合は再読込のため npm run dev コマンドを再実行してください。
- ブラウザに表示される画像はローカルの画像ではなく image.upspash.com に保存されている画像が表示されます。

# SEO メタタグの設定
>https://alpacat.com/blog/nextjs132-metadata-api/
>[App RouterのOGP設定方法まとめ [Next.js]](https://zenn.dev/temasaguru/articles/641a10cd5af02a)

検索エンジンに公開したページを見つけ上位表示してもらうためにはメタタグの設定を行う必要があります。

!!! info `Next.js 13`では、メタデータでタイトル・タグなどを設定する
```typescript
const siteName= 'サイト名';
const description = 'サイトの説明';
const url = 'https://本番のドメイン';

export const metadata = {
  title: {
    default: siteName,
    /** `next-seo`の`titleTemplate`に相当する機能 */
    template: `%s - ${siteName}`,
  },
  description,
  openGraph: {
    title: siteName,
    description,
    url,
    siteName,
    locale: 'ja_JP',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: siteName,
    description,
    site: '@サイト用アカウントのTwitterID',
    creator: '@作者のTwitterID',
  },
  verification: {
    google: 'サーチコンソールのやつ',
  },
  alternates: {
    canonical: url,
  },
}
```
ブラウザのタブには設定した title が表示されていることを確認できます。
# faviconとOG画像はファイルを設置するだけでいい
>[公式](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/app-icons)

app 直下に、 icon.(ico|jpg|jpeg|png|svg) と apple-icon.(jpg|jpeg|png|svg) というファイル名で画像を設置してください。
- 結果
```html
<link rel="icon" href="/favicon.ico" sizes="any" />
<link rel="icon" href="/icon?<generated>" type="image/<generated>" sizes="<generated>">
<link rel="apple-touch-icon" href="/apple-icon?<generated>" type="image/<generated>" sizes="<generated>">
```
画像を適切なファイル名で置いただけで、faviconのメタタグが設定されます。
# OG画像の設置方法
>[公式](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/opengraph-image)

app 直下に、 opengraph-image.(jpg|jpeg|png|gif) というファイル名で画像を設置してください。
- 結果
```html
<meta property="og:image" content="<generated>" />
<meta property="og:image:type" content="<generated>" />
<meta property="og:image:width" content="<generated>" />
<meta property="og:image:height" content="<generated>" />
```
画像を適切なファイル名で置いただけで、OG画像のメタタグが設定されます。 これなら画像の絶対URLを考える必要が一切なくなります。

>https://zenn.dev/temasaguru/articles/2968736b5a2f41

なお、opengraph-image.tsx を設置すれば、zennや質問箱のような動的OG画像を生成できます。
- 詳しくは上記の記事をご覧ください。
# サイトURLの設定
`metadata.metadataBase is not set for resolving social open graph or twitter images, fallbacks to...`

上記の設定ではmetadataBaseを書いていないため、ローカル環境でこのような警告が出るはずです。
```typescript
export const metadata = {
  // Netlifyなので指定が必要 https://docs.netlify.com/configure-builds/environment-variables/
  metadataBase: new URL(process.env.URL ?? 'http://localhost:3000'),
 // 後略
```
もしVercel以外にデプロイする場合は、metadataBaseにURLオブジェクトを設定してください。
- 直書きしてしまうと、プレビューデプロイのOG画像がプロダクションに向いてしまうといった問題が発生するため、ホスティングサービスが用意する環境変数を使うことを推奨します。
- なぜVercelでは必要ないかというと、https://${process.env.VERCEL_URL}にフォールバックするからです。

>https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadatabase





















