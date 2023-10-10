- [Next.js(App directory) + Jest](#nextjsapp-directory--jest)
  - [セットアップ](#セットアップ)
- [テストディレクトリを作成](#テストディレクトリを作成)
- [テストの作成](#テストの作成)
  - [関数テスト](#関数テスト)
  - [UIテスト](#uiテスト)
- [テスト実行](#テスト実行)
  - [結果](#結果)
  - [エラーが発生した場合](#エラーが発生した場合)


# Next.js(App directory) + Jest
>[Next.js(App directory) + Jestを使ったハンズオン](https://developer-souta.com/blog/bm6k91h7xw9j)
>[Next.jsのプロジェクトにJestを導入する](https://dabohaze.site/next-js-jest/)

環境構築から簡単なテストまで

## セットアップ
>https://nextjs.org/docs/pages/building-your-application/optimizing/testing#setting-up-jest-with-the-rust-compiler
- プロジェクトルートで実行
```bash
npm install --save-dev jest jest-environment-jsdom @testing-library/react @testing-library/jest-dom
```
そしてJest用の設定ファイルの作成
```bash
touch jest.config.mjs
```
./jest.config.mjs
```js
import nextJest from 'next/jest.js';

const createJestConfig = nextJest({
  // Provide the path to your Next.js app to load next.config.js and .env files in your test environment
  dir: './',
});

// Add any custom config to be passed to Jest
/** @type {import('jest').Config} */
const config = {
  // Add more setup options before each test is run
  // setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],

  testEnvironment: 'jest-environment-jsdom',
};

// createJestConfig is exported this way to ensure that next/jest can load the Next.js config which is async
export default createJestConfig(config);
```
npm run test でテストを実行することが出来るようにスクリプトも追加

package.json
```json
"scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    // 追加
     "test": "jest --watch"
    },
```
# テストディレクトリを作成
```bash
# 統合テスト用のディレクトリ
mkdir src/app/test
```
# テストの作成
## 関数テスト
引数で受け取った数値を2倍して返す単純な関数です。
>./src/functions/hoge.ts
```ts
export const hoge = (x: number): number => { return x * 2;
};
```
- テストコード
>./src/app/test/hoge.test.ts
```ts
import { hoge } from '@/functions/hoge';

describe('hoge', () => {
  it('hoge関数の引数に1を渡したら2が返ってくること', () => {
    expect(hoge(2)).toBe(4);
  });
});
```
- テスト実行
```bash
$ npm run test
 PASS  __tests__/hoge.test.ts
  hoge
    ✓ hoge関数の引数に1を渡したら2が返ってくること (1 ms)
```

## UIテスト
./src/app/test/page.test.tsx
```ts
import Home from '@/app/page';
import '@testing-library/jest-dom';
import { render, screen } from '@testing-library/react';


describe('Home', () => {
  it('renders a heading', () => {
    render(<Home />);

    const heading: HTMLElement = screen.getByRole('heading', {
      name: /Deploy/i,
    });

    expect(heading).toBeInTheDocument();
  });
});
```
これはホームコンポーネントをレンダリングしてその中のhタグ内にDeployという文字が含まれているかどうかというテストである。
公式ドキュメントではwelcome to next.jsという文字が含まれているテストとなっているが最新版のNext.jsでは含まれていないためpage.tsxのh2タグの他の文字から代入している。
また、headingというロールがあるがこれはしっかりとhタグ関連の検知という認識がなければ解決が難しくなるのでしっかりとドキュメントを読む癖をつくなければいけない。

# テスト実行
`npm run test`
`yarn test`
- 全てのテストが実行される

`npm run test hoge.test.js`
`yarn test hoge.test.js`
- ファイル名を指定すると、指定したファイルのテストのみ実行されます。



## 結果
```bash
$ npm run test
 PASS  __tests__/page.test.tsx
  Home
    ✓ renders a heading (141 ms)


Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        2.714 
```
となっていれば成功である。

## エラーが発生した場合

Error: Jest: Failed to parse the TypeScript config file /Users/tomokiohara/Develop/nextjs/my-app/jest.config.ts
Error: Jest: 'ts-node' is required for the TypeScript configuration files. Make sure it is installed
もしテストを実行して上記のようなエラーが発生した場合は、ts-nodeをインストールする。

```bash
npm install ts-node
OR
yarn add ts-node
```


