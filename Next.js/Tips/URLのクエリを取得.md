# URLのクエリパラメーターを取得する
>[【React】Next.js13以降でURLのクエリパラメーターを取得する方法](https://qiita.com/someone7140/items/f45b38c0fe27ae0bac87)
>[Next.js 13 で URL の情報を取得する方法を整理する 【useRouter・useSearchParams・usePathname・useParams】(next/navigation)](https://qiita.com/RANZU/items/0037cbb04d8716944b0e)
>https://zenn.dev/yumemi_inc/articles/next-13-app-overview#userouter-%E3%81%AE%E5%A4%A7%E5%B9%85%E5%A4%89%E6%9B%B4

URL を clothes や shoes に変更しても同じ”商品のページです”が表示されるので/products/以下の URL に入っている文字列をページ内容に表示させるため`useSearchParams・usePathname・useParams`を利用します。
- これらを利用することでアクセスしてきた URL によって動的にページの内容を変えることができます。

# `useSearchParams`
ページにアクセスするときに付与したクエリパラメータが取得できる
```typescript
import { useSearchParams } from 'next/navigation';

// 省略

  const searchParams = useSearchParams()
  console.log(searchParams)
  
// 省略
```
## 実行結果
```bash
# /demo/hoge?params1=fuga にアクセスした結果

params1=fuga
```
# usePathname
アクセスしたときのパスが取得できる
- このパスは，dynamic params を含みます。
```typescript
"use client";
import { usePathname } from 'next/navigation';
export default function Name() {
  const pathname = usePathname()
  const name = pathname.split('/')[2]
  console.log(pathname)
  console.log(name)
  return (
    <div>This is the {name} page</div>
  )
}
```
## 結果
```bash
# /demo/hoge?params1=fuga にアクセスした結果
params1=fuga
```
# `useParams`
dynamic params を取得
```typescript
import { useParams } from 'next/navigation';

// 省略

  // useParams の戻り値はオブジェクト
  const params = useParams()
  console.log("result of useParams: %o", params)
  
// 省略
```
最後に，`useParams` の実行結果です。dynamic params である [id] が取得できていることが確認できました。
## 実行結果
```bash
# /demo/hoge?params1=fuga にアクセスした結果

{id: 'hoge'}
```




