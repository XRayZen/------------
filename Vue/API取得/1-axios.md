
# axios
- npmでインストール
- `npm install axios`


>[vue.jsを使ってaxiosを学ぶ](https://reffect.co.jp/vue/vue-axios-learn/#axios-%E3%81%AE%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95)

# GET メソッドによるデータ取得
GET メソッドは一番シンプルなメソッドで axios のみならず外部から情報を取得する際の基本になる HTTP のリクエストメソッドの一つです。ブラウザから WEB サーバにアクセスする際も GET メソッドを利用して WEB ページに表示するために必要な情報を取得しています。

JSONPlaceholder には GET メソッドで取得することができる 6 つのリソースが準備されています。一番データ取得数の少ない users から情報を取得してみましょう。

ブラウザからhttps://jsonplaceholder.typicode.com/usersにアクセスしても users のデータを確認することができるので axios を利用する前にどのような情報が取得できるかを確認することができます。本文書では id と name のみ注目します。

axios の GET メソッドの基本書式は下記のとおりです。get の引数に URL を入れるだけで URL に対して GET リクエストを送ることができます。リクエスト後に戻される値はすべて response の中に保存されます。
```javascript
axios
  .get('/user?ID=12345')
  .then(function (response) {
    // handle success(axiosの処理が成功した場合に処理させたいことを記述)
    console.log(response);
  })
  .catch(function (error) {
    // handle error(axiosの処理にエラーが発生した場合に処理させたいことを記述)
    console.log(error);
  })
  .finally(function () {
    // always executed(axiosの処理結果によらずいつも実行させたい処理を記述)
  });
```
```javascript
// Arrow関数を利用
axios
  .get('/user?ID=12345')
  .then((response) => {
    // handle success(axiosの処理が成功した場合に処理させたいことを記述)
    console.log(response);
  })
  .catch((error) => {
    // handle error(axiosの処理にエラーが発生した場合に処理させたいことを記述)
    console.log(error);
  })
  .finally( () =>  {
    // always executed(axiosの処理結果によらずいつも実行させたい処理を記述)
  });
```
async, await を利用した場合は上記の基本書式と記述方法が変わり、try,catch を利用します。

書式
```javascript
try {
  // handle success(axiosの処理が成功した場合に処理させたいことを記述)
  const response = await axios.get('/user?ID=12345');
  console.log(response.data);
} catch (error) {
  // handle error(axiosの処理にエラーが発生した場合に処理させたいことを記述)
  console.log(error);
} finally {
  // always executed(axiosの処理結果によらずいつも実行させたい処理を記述)
  console.log('message');
}
```
await を利用した場合は async が必要となるので注意してください。
- Vue インスタンスの中で axios の GET メソッドを利用します。mounted の中に axios の実行コードを記述しているため、ブラウザでこのファイルを閲覧すると axios が実行されます。console.log で response を指定しているのでデベロッパーツールを使って実行結果を確認することができます。



