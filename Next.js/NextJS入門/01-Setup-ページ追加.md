# Next.js 13 with TypeScriptの入門　
2023年5月11日 更新
最新のNext.js13に合わせて改変
>https://reffect.co.jp/react/next-js/

- [Next.js 13 with TypeScriptの入門　](#nextjs-13-with-typescriptの入門)
- [Next.js のインストール](#nextjs-のインストール)
  - [Next.js 開発サーバの起動](#nextjs-開発サーバの起動)
  - [next.js のデフォルトページ](#nextjs-のデフォルトページ)
- [Hello Next.js](#hello-nextjs)
  - [ディレクトリ構成](#ディレクトリ構成)
    - [public ディレクトリ](#public-ディレクトリ)
  - [index.js ファイルの更新(Fast Refresh)-ホットリロード](#indexjs-ファイルの更新fast-refresh-ホットリロード)
    - [ソースを見て動作を確認](#ソースを見て動作を確認)
    - [React と比較して違い](#react-と比較して違い)
    - [レンダリング方式の違い](#レンダリング方式の違い)
  - [別のページを作成する](#別のページを作成する)
  - [404ページ](#404ページ)


# Next.js のインストール
実際に公式ドキュメントを参考に手を動かしながら説明を進めていくため最初に Next.js のプロジェクトの作成を行います。プロジェクト名は任意の名前をつけることができるのでここでは next-js-13 としています。npx create-next-app@latest コマンドを実行すると TypeScript などの機能 をプロジェクトで利用するかどうか聞かれますがすべてデフォルトの値を選択しています。 App Router を利用するかどうかも選択することができますが recommeded と表示されているため新しくプロジェクトを作成する場合に App Router を利用することが推奨されていることがわかります。
- [create-next-appで訊かれていること](https://zenn.dev/ikkik/articles/51d97ff70bd0da)を参照
-  ES7 React/Redux/GraphQL/React-Native snippets の Extention がおすすめ
```bash
 % npx create-next-app@latest
Need to install the following packages:
  create-next-app@13.4.1
Ok to proceed? (y) y
✔ What is your project named? … next-js-13
✔ Would you like to use TypeScript with this project? … No / Yes
✔ Would you like to use ESLint with this project? … No / Yes
✔ Would you like to use Tailwind CSS with this project? … No / Yes
✔ Would you like to use `src/` directory with this project? … No / Yes
✔ Use App Router (recommended)? … No / Yes
✔ Would you like to customize the default import alias? … No / Yes
Creating a new Next.js app in /Users/mac/Desktop/next-js-13.
```
プロジェクトの名前を聞かれるので任意のプロジェクト名を入力してください。
デフォルトの名前のまま進む場合は Enter を押します。
コマンドを実行したディレクトリに指定した任意の名前もしくはデフォルトの my-app ディレクトリが作成されます。

!!! info src ディレクトリの作成はデフォルトでは”No"になっていますが"Yes"を選択すると src ディレクトリの下に app ディレクトリが作成されます。
    "No"の場合はプロジェクトディレクトリ直下に app ディレクトリが作成されます。
     - default import alias を利用するとコンポーネントを import する際に"../../components/User"といったコードを"@/components/User" というように記述することが可能になります。

## Next.js 開発サーバの起動
インストールが完了するとコマンドを実行した場所にプロジェクト名のディレクトリが作成されます。

プロジェクトディレクトリ my-app に移動して開発サーバを起動するために npm run dev コマンドを実行します。
コマンド実行後のメッセージを確認すると http://localhost:3000 でサーバが起動していることがわかります。

Terminal
```bash
 % cd my-app
 % npm run dev

> my-app-next@0.1.0 dev
> next dev
ready - started server on 0.0.0.0:3000, url: http://localhost:3000
event - compiled client and server successfully in 2.1s (172 modules)
```
ブラウザを起動して http://localhost:3000 にアクセスすると Next.js の初期画面が表示されます。
これで Next.js のインストールは完了です。
## next.js のデフォルトページ

プロジェクトの中にある package.json ファイルを確認すると開発中に利用することができるコマンド(npm run dev など)やインストールした Next.js のバージョンを確認することができます。
```json
{
  "name": "my-app-next",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "12.3.0",
    "react": "18.2.0",
    "react-dom": "18.2.0"
  },
  "devDependencies": {
    "eslint": "8.23.1",
    "eslint-config-next": "12.3.0"
  }
}
```
![Alt text](https://reffect.co.jp/images/react/next-js-13/next-js-13-4-1-1024x718.webp)

# Hello Next.js
## ディレクトリ構成
プロジェクト直後のディレクトリ構成の確認を行います。
プロジェクト作成直後から存在するディレクトリは src/app, styles, node_modules, public の 4 つです。
- その他に `README.md` と package.json と package-lock.json の 3 つのファイルが存在します。
- 先頭に.(ドット)が含まれる.next ディレクトリ、.gitignore ファイルもあります。

ブラウザで表示されたページの内容は pages ディレクトリの中の index.js ファイルに記述されています。このディレクトリの中にアプリケーションのコアとなるコードを記述していくことになります。

>Next.js ディレクトリ構成
![Alt text](https://reffect.co.jp/images/react/next-js/next-js-directory-1024x449.webp)

### public ディレクトリ
public ディレクトリの中には、favicon.ico, vercel.svg ファイルが保存されています。
- これらのファイルはブラウザから直接アクセスすることができます。

ブラウザの URL の http://localhost:3000 の後ろに/vercel.svg を追加してください。
- Vercel のロゴがブラウザ上に表示されます。public ディレクトリに保存したファイルには直接ブラウザからアクセスできることが確認できました。
- このことから public には静的なファイルを保存できることがわかります。

!!! example 例えば about.html ファイルを public ディレクトリに作成するとブラウザから直接アクセスすることができ、about.html ファイルの内容を表示させることも可能です。

styles ディレクトリの下には CSS のファイルを保存します。
- CSS ファイルについては public ディレクトリ下に css ファイルを作成して利用することもできます。
- public ディレクトリを利用した場合は JavaScript のバンドルとしてではなく link タグで直接ファイル名を指定することで CSS を適用することになります。

## index.js ファイルの更新(Fast Refresh)-ホットリロード
pages ディレクトリにある index.js ファイルを更新するとブラウザ上に表示される内容も変更されるのか確認を行います。

index.js の中身を一度削除して下記のように更新します。
```javascript
export default function Home() {
  return <h1>Hello Next.js</h1>;
}
```
!!! info React の場合はファイル上部で import React from 'react'を記述しますが、Next.js では必要ありません。
    React においてもバージョンがアップし React の import が必須ではなくなりました。

npm run dev を実行していれば index.js を更新するとブラウザに自動で更新内容が反映されます。
- これは Next.js が持つ Fast Refresh という機能です。
  - ページをリロードすることなく更新が即反映されるので開発者の効率化につながる非常に便利な機能です。

![Alt text](https://reffect.co.jp/images/react/next-js/next-js-index-js-1024x648.webp)

### ソースを見て動作を確認
ブラウザに表示されている内容だけではなく Next.js のページ表示に関する動作を確認するためにページのソースを確認してみましょう。ブラウザ上で右クリックを行いページのソースを表示してください。

![Alt text](https://reffect.co.jp/images/react/next-js/next-js-hello-source-1024x476.webp)

ページのソースの中に”Hello Next.js”の文字列が入っていることがわかります。
- index.js は JavaScript ファイルなので通常では JavaScript ファイルをブラウザが受け取って JavaScript ファイルを処理してその内容を画面に描写します。

ブラウザのページのソースを見る限りブラウザが直接 HTML の情報(h1 タグの Hello World)を受け取っていることがわかります。
- これは Next.js ではブラウザに送信する前にサーバ側で Pre-Rendering を行っているからです。
- HTML の情報をそのまま受け取っているので”Hello Next.js”を表示するためにブラウザ側で JavaScript の処理を行う必要がありません。
  - デフォルトでは Next.js はすべてのページで Pre-Rendering を行います。
  
!!! info Pre-Rendering とはクライアント側の JavaScript で処理を行う前に Next.js 側(サーバ)でページを事前に作成し作成したページをクライアントに送信する機能です。

しかしこれだけの情報では Pre-Rendering が行われているかはわかりにくいと思うので React と比較して違いを見てみましょう。

### React と比較して違い
React では App.js ファイルに h1 タグで”Hello React”と記述しています。
- ブラウザで”Hello React”が表示されていることを確認し、ページのソースを確認します。

下の画像は字が細かすぎて見えないかもしれませんが React の場合はページのソースのどこにも h1 タグは表示されていません。
![Alt text](https://reffect.co.jp/images/react/next-js/react-hello-source-1024x666.webp)

!!! question つまり React ではブラウザが受け取った情報には h1 タグが記述されておらず受信した JavaScript ファイルをブラウザが処理することで h1 タグとその内容を描写していることがわかります。

!!! info ブラウザ(=クライアント)側で処理を行うため Client Side Rendering(クライアントサイドレンダリング)と呼ばれます。

### レンダリング方式の違い
ブラウザのページのソースを比較することで Client Side Rendering と Pre-Rendering の違いを理解することができました。

Next.js では Pre-Rendering の方法に 3 つの方法があります。
- Static Site Generation(SSG)
- Incremental Static Regeneration(ISR)
- Server Side Rendering(SSR)の 3 つ

本文書でもそれぞれのレンダリングについて確認していきます。

## 別のページを作成する
>[Next.js13最新バージョンの参考](https://udemy.benesse.co.jp/development/next-js13.html)

App Router と呼ばれる新機能でページ追加・ルーティング・レンダリング・データ取得(Fetch)・キャッシュが変更された
>[Next.js 13 app directory で記事投稿サイトを作ってみよう](https://zenn.dev/azukiazusa/articles/next-js-app-dir-tutorial)

!!! info 従来のNext.jsではプロジェクトの作成時に「pages」という名称のディレクトリが作成されていました。Next.js 13でappディレクトリの機能を使用する場合、「pages」ではなく「app」という名称のディレクトリが必要です。
    ただし、プロジェクトの新規作成時にappディレクトリの使用を選択していれば、「app」ディレクトリが自動で作成されます。
    pagesディレクトリからappディレクトリへの移行作業は不要です。

Next.js 13でプロジェクトに新規ページを追加する場合、appディレクトリ内に新たなディレクトリを作成し、その中に「page.tsx」ファイルを作成します。

例えば「/test」というページを作る際は、appディレクトリ内に「test」という名称のディレクトリを作成し、「page.tsx」ファイルに以下のコードを記述しましょう。

!!! error ファイル名が`page.tsx`でないと動作しない
```typescript
// Arrow function
const Test = () => {
  return &lt;p>Testページ&lt;/p>;
};
 
export default Test;
```
- またはこの方法でもいける
```typescript
export default function About() {
    return <h1>About Page</h1>;
}
```
Next.js を利用するとルーティングの設定が自動で行われるため簡単にページを作成することができることが確認できました。

![Alt text](https://udemy.benesse.co.jp/wp-content/uploads/29d3bc99193cb0b837fb8cc53f050ae5-2.jpg)

!!! tip Visual Sdutio Code で ES7 React/Redux/GraphQL/React-Native snippets をインストールしている場合は rafce または rcfe を打った後に Tab キーを押すと関数のテンプレートが表示されます。
## 404ページ
ここまでの設定では"/"(ルート)と/about ページ以外にアクセスを行うと 404 エラーが表示されます。

!!! info 404 はページが存在しないページにアクセスした場合に表示される HTTP のステータスコードで"NOT FOUND"を意味します。

