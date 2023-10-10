
>https://reffect.co.jp/react/next-js-13/#route-handlers

# Route Handlers
App Router の Route Handlers は Pages Router の API Routes と同等の機能です。
- Route Handlers は API エンドポイントとして利用することができ、GET, POST などの HTTP メソッドを利用してアクセスすることができます。

## 実践
Route Handlers も app ディレクトリの中に任意の名前のディレクトリを作成して設定を行います。ここでは api という名前のディレクトリを作成します。
- Page コンポーネントの名前が page.tsx で決められているように Route Handlers では route.js または route.ts という名前をつける必要があります。

## GET リクエストの動作確認
api ディレクトリに route.ts ファイルを作成して以下のコードを記述します。

>route.ts
```typescript
import { NextResponse } from 'next/server';

export function GET() {
  return NextResponse.json({ name: 'John Doe' });
}
```
/api に対して GET リクエストを送信すると JSON で{"name":"John Doe"}が戻されるというもっともシンプルなコードです。

GET であればブラウザからアクセスすることで動作確認できます。
![Alt text](https://reffect.co.jp/images/react/next-js-13/next-js-13-4-34-1024x453.webp)

# Route Handlers からのデータ取得
Route Handlers の中から JSONPlaceHolder にアクセスを行いデータを取得することもできます。

>router.ts
```typescript
import { NextResponse } from 'next/server';

export async function GET() {
  const response = await fetch('https://jsonplaceholder.typicode.com/users');
  const data = await response.json();
  return NextResponse.json(data);
}
```
ブラウザからでも/api でアクセスすることができますが UserList コンポーネントからアクセスを行ってみます。

>UserList.tsx
```typescript
type User = {
  id: string,
  name: string,
  email: string,
};

const UserList = async () => {
  // await new Promise((resolve) => setTimeout(resolve, 5000));
  const response = await fetch('http://localhost:3000/api');
  if (!response.ok) throw new Error('Failed to fetch data');
  const users: User[] = await response.json();
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};

export default UserList;
```
ブラウザから/users にアクセスすると Route Hadlers を経由してデータの取得が行われ、ユーザ一覧が表示されます。
# URL パラメータの取得
検索など URL パラメータを利用した場合の URL パラメータの取得方法を確認します。

UserLists.tsx ファイルでは fetch 関数で指定する URL にパラメータを追加します。App Router で fetch 関数は Web API の fetch 関数を拡張しているためオプションを設定することができます。デフォルトでは一度 fetch 関数が実行されるとキャッシュされるためその後 fetch 関数を実行するとキャッシュしたデータが利用されるためリクエストが行われません。ここでは動作確認のためキャッシュ機能を無効にします。

UserList.tsx
const response = await fetch('http://localhost:3000/api?name=John', {
  cache: 'no-store',
});
api/route.ts では URL パラメータを取得するため request オブジェクトを利用します。request オブジェクトにはさまざまな情報が含まれていますが URL パラメータは request.url を利用して取得します。

api/route.ts
import { NextResponse } from 'next/server';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const name = searchParams.get('name');
  console.log(name);
  const response = await fetch('https://jsonplaceholder.typicode.com/users');
  const data = await response.json();
  return NextResponse.json(data);
}
ブラウザから/users にアクセスすると開発サーバを起動したターミナルに"John"が表示されます。URL パラメータを取得することができるようになりました。

headers, cookies 関数
headers, cookies 関数を利用することでそれらの情報を取得することができます。

api/route.ts
import { NextResponse } from 'next/server';
import { headers, cookies } from 'next/headers';

export async function GET() {
  const headersList = headers();
  const cookieStore = cookies();

  console.log('headersList', headersList);
  console.log('cookieStore', cookieStore);

  const response = await fetch('https://jsonplaceholder.typicode.com/users');
  const data = await response.json();
  return NextResponse.json(data);
}
Dynamic Routes の設定
api のように静的な API のエンドポイントではなく api/1, api/2,...api/100 などのように URL が動的に変わる場合の設定方法を確認します。

api ディレクトリの下に[]で囲んだ[id]ディレクトリを作成してその下に route.ts ファイルを作成します。

api/route.ts
import { NextResponse } from 'next/server';

export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const id = params.id;
  return NextResponse.json(id);
}
ブラウザから/api/100 にアクセスすると"100"が戻されます。

実際のアプリケーションではこの値を利用してデータベースにアクセスしてレコードを取得したり、さらに別のサーバにアクセスを行いデータを取得してページを表示するといった設定を行います。

POST の設定
POST リクエストによって送信されてきたデータを取得する方法を確認します。

api/route.ts ファイルで POST リクエストで送信されてきたデータを取り出すためのコードを追加します。関数の名前は POST となります。request.json()で取得したデータをそのままクライアントに戻しています。通常はデータベースなどへのデータの挿入などを行います。

api/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const response = await fetch('https://jsonplaceholder.typicode.com/users');
  const data = await response.json();
  return NextResponse.json(data);
}

export async function POST(request: Request) {
  const res = await request.json();
  return NextResponse.json({ res });
}
POST リクエストを送信するためには入力フォームを作成する必要がありますがここでは fetch 関数を利用して POST リクエストでデータを送信するコードのみ users/page.tsx ファイルに追加します。

page.tsx
import UserList from './UserList';

const Page = async () => {
  const response = await fetch('http://localhost:3000/api', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      name: 'John',
      email: 'john@example.com',
    }),
  });

  const data = await response.json();

  console.log(data);

  return (
    <div className="m-4">
      <h1 className="text-lg font-bold">ユーザ一覧</h1>
      {/* @ts-expect-error Async Server Component */}
      <UserList />
    </div>
  );
};

export default Page;
/users にアクセスを行い、body プロパティに送信したオブジェクトの中身が開発サーバを起動しているターミナルに表示されれば Route Handlers で正しく POST リクエストを受け取り、受け取ったデータをブラウザに戻していることになります。

以下のように表示されれば正しく動作しています。

Terminal
{ res: { name: 'John', email: 'john@example.com' } }
実際のアプリケーションでは POST リクエストから送信されてきたデータをデータベースに登録するといった処理を行います。




