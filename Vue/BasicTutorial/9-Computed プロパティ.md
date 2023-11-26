- [Computed プロパティ](#computed-プロパティ)
  - [Computed プロパティを利用しない場合](#computed-プロパティを利用しない場合)
  - [Computed プロパティを利用した場合](#computed-プロパティを利用した場合)
  - [Computed プロパティ 中で filter 関数を利用](#computed-プロパティ-中で-filter-関数を利用)
- [Computed vs Function](#computed-vs-function)
  - [Computed プロパティのキャッシュ機能](#computed-プロパティのキャッシュ機能)
  - [どのタイミングで実行されている確認](#どのタイミングで実行されている確認)
- [Computed セッター](#computed-セッター)
  - [Setter を利用することで Compute プロパティを更新](#setter-を利用することで-compute-プロパティを更新)


# Computed プロパティ
Computed プロパティは定義済みの変数を利用して計算、加工を行うことで元のデータとは異なる形でユーザに表示することができる機能
- reactive な変数を利用して Computed プロパティを定義した場合は reactive な変数が更新されるとその更新に合わせて再計算、再加工が自動で行われる

Computed プロパティを利用しない場合とした場合を比較して Computed プロパティの理解を深めていきます。
## Computed プロパティを利用しない場合
reactive 関数を利用して user を定義して firstName と lastName を template タグに設定してブラウザ上に表示させます。
```html
<script setup>
  import { reactive } from 'vue';
  const user = reactive({
    firstName: 'John',
    lastName: 'Doe',
  });
</script>

<template>
  <h1>Vue 3 入門</h1>
  <h2>fullName: {{ user.firstName }} {{ user.lastName }}</h2>
</template>
```
ブラウザ上には設定通りに fullName が表示されます。
##  Computed プロパティを利用した場合
これを computed プロパティを使って表示させます。
- computed プロパティを利用するためには vue から computed 関数を import する必要があります。
- import した computed 関数の引数に関数を記述します。関数の戻り値が template タグの中で設定した fullName に表示されます。
  - 表示される内容は computed プロパティを使う前と変わりません。
```html
<script setup>
  import { reactive, computed } from 'vue';
  const user = reactive({
    firstName: 'John',
    lastName: 'Doe',
  });

  const fullName = computed(() => `${user.firstName} ${user.lastName}`);
</script>

<template>
  <h1>Vue 3 入門</h1>
  <h2>fullName: {{ fullName }}</h2>
</template>
```
Computed プロパティは定義済みの変数を利用して加工して表示することができると説明した通りの動作になっていることが確認できました。

この例はシンプルなので Computed プロパティを利用することによる恩恵は少ないかもしれませんが

!!! success fullName という名前を定義することでどのような処理を行なっているかもすぐにわかります
    また何度も template 内で利用したい場合に computed プロパティの fullName と記述したほうがコードも短い上、
      - 表示の内容を変更したい場合(すべての文字を大文字にするなど)も
      - fullName の中身を 1 箇所変更することで対応することができます。

!!! warning script タグで computed プロパティにアクセスしたい場合は ref 関数と同様に value をつける必要があるので注意してください。

## Computed プロパティ 中で filter 関数を利用
リストと分岐の動作確認では v-for ディレクティブと v-if ディレクティブを利用して users 情報から admin が true のユーザのみ表示する設定を確認しましたが Computed プロパティを利用することで admin ユーザのみ表示させることができます。

Computed プロパティ adminUsers の中で filter 関数を利用して admin が true のユーザのみ取得します。
- v-for ディレクティブの中では Computed プロパティ adminUsers を利用して繰り返し処理を行っています。
```html
<script setup>
  import { computed } from 'vue';
  const users = [
    { id: 1, name: 'John Doe', email: 'john@test.com', admin: true },
    { id: 2, name: 'Jane Doe', email: 'jane@example.com', admin: false },
    { id: 3, name: 'Kevin MacDonald', email: 'kevin@test.com', admin: false },
  ];

  const adminUsers = computed(() =>
    users.filter((user) => user.admin === true)
  );
</script>

<template>
  <h1>Vue 3 入門</h1>
  <div v-for="user in adminUsers" :key="user.id">
    <div>{{ user.id }} {{ user.name }} {{ user.email }}</div>
  </div>
</template>
```
admin の値が true のユーザのみ表示されます。
- 定義済みの変数を Computed プロパティで加工することでユーザに表示しています。
- adminUsers という名前をつけているので名前からどのような情報を設定していることも理解することができます。

filter 関数内で条件式を変更するだけで admin ではないユーザを取得することができます。
```javascript
const generalUsers = computed(() => users.filter((user) => user.admin ===
false));
```

# Computed vs Function
Computed プロパティで記述した内容は関数を使っても以下のように記述することができます。
- マスタッシュの中で関数を利用する場合は Computed プロパティと異なり関数の名前の後ろに()が必要です。
- Computed プロパティと関数で同じことができるのであればどちらの機能を利用する必要がありません。
```html
<script setup>
  import { reactive, computed } from 'vue';
  const user = reactive({
    firstName: 'John',
    lastName: 'Doe',
  });

  const fullName = () => `${user.firstName} ${user.lastName}`;
</script>

<template>
  <h1>Vue 3 入門</h1>
  <h2>fullName: {{ fullName() }}</h2>
</template>
```
## Computed プロパティのキャッシュ機能
しかし Computed プロパティと関数には大きな違いがあり、Computed プロパティはキャッシュ機能を持っています。
- キャッシュ機能がどのようなものか Computed プロパティと関数を利用して確認を行っていきます。

関数と Computed プロパティを同時に実行するために新たに Computed プロパティの cFullName 関数を追加します。
```javascript
const fullName = () => `${user.firstName} ${user.lastName}`;
const cFullName = computed(() => `${user.firstName} ${user.lastName}`);
```
Computed プロパティのキャッシュ機能を確認するために Math.random()関数は追加します。
- Math.random()関数は実行するとランダムな数字を戻すという関数です。
```javascript
const fullName = () => `${Math.random()} ${user.firstName} ${user.lastName}`;
const cFullName = computed(
  () => `${Math.random()} $${user.firstName} ${user.lastName}`
);
```
キャッシュ機能を確認するために tempalte タグの中で複数の関数と computed プロパティを実行させます。
```html
<template>
  <h1>Vue 3 入門</h1>
  <h2>fullName: {{ fullName() }}</h2>
  <h2>fullName: {{ fullName() }}</h2>
  <h2>fullName: {{ fullName() }}</h2>
  <h2>cFullName: {{ cFullName }}</h2>
  <h2>cFullName: {{ cFullName }}</h2>
  <h2>cFullName: {{ cFullName }}</h2>
</template>
```

開くと下記のように fullName 関数ではすべての fullName 関数の実行で Math.randam 関数が実行されるため異なる値が表示されていますが Computed プロパティの cFullName はキャッシュ機能を持っているため一度 Math.randam 関数が実行されるだけで後は同じ値を持っていることがわかります。

Computed プロパティは reactive な変数の更新を反映する際に再実行されるので input 要素の v-model ディレクティブに user.firstName を設定してどのような変化が起こるか確認します。
```html
<input v-model="user.firstName" />
```
文字を入力、削除する度に値が変更になりますが Computed プロパティの Math.randam の値はすべての cfullName で一緒であることがわかります。
- fullName は文字を変更する度にすべての fullName で Math.random が実行されています。
## どのタイミングで実行されている確認
さらにキャッシュ機能を確認するために Computed プロパティと関数に console.log を設定してどのタイミングで実行されている確認します。
- 実行された場合にはデベロッパーツールに指定した文字列が表示れます。
```javascript
const fullName = () => {
  console.log('Function');
  return `${user.firstName} ${user.lastName}`;
};
const cFullName = computed(() => {
  console.log('Computed Property');
  return `${user.firstName} ${user.lastName}`;
});
```
定義した reactive な変数の user の firstName を変更するとどちらも再実行されることは確認済みなので別の reactive な変数 count を ref 関数で定義します。
- 定義した count の数を click イベントを利用して更新します。
```html
<script setup>
import { ref, reactive, computed } from 'vue';
const count = ref(0);

const user = reactive({
  firstName: 'John',
  lastName: 'Doe',
});

const fullName = () => {
  console.log('Function');
  return `${user.firstName} ${user.lastName}`;
};
const cFullName = computed(() => {
  console.log('Computed Propety');
  return `${user.firstName} ${user.lastName}`;
});
</script>
```
ページを開くとコンソールには”Function" と "Computed Property”のメッセージが表示されます。
- その後ボタンをクリックすると Function のみメッセージが表示されます。
- 下記の画像では Click を 6 回行った場合に Functions の左側に 6 が表示されていることが確認できます。
  - Computd プロパティは Computed プロパティに関連のない reactive の変数の影響をうけないことがわかりました。
![Alt text](https://reffect.co.jp/images/vue/beginner-vue/vue3-beginner-34-1024x568.webp)

!!! info Computed プロパティと関数で同じ処理ができたとしても違いを理解した上でどちらを利用するか決める必要があります。

!!! info 関数とComputed プロパティの違い
    関数はデータが変更されるたびに呼び出されます。
    - 関数は、単純な計算や条件付きの計算に適しています。

    Computedプロパティは、依存関係を持つデータを計算するために使用されます。C
    - 依存関係が変更された場合にのみ再評価されます。
      - つまり、依存関係が変更されない限り、Computedプロパティは再評価されません。
    - Computedプロパティは、複雑な計算や、複数のデータに基づく計算に適しています。

# Computed セッター
script タグの中では computed プロパティの値にアクセスする場合には.value が必要であることを説明しました。
- .value を利用して Computed プロパティを更新することができるのか確認します。

click イベントを持つボタンを追加してボタンをクリックすると changeName 関数を実行するように設定します。
- changeName 関数では.value を使って別の名前に更新を行います。
```html
<script setup>
import { reactive, computed } from 'vue';

const user = reactive({
  firstName: 'John',
  lastName: 'Doe',
});

const fullName = computed(() => `${user.firstName} ${user.lastName}`);

const changeName = () => {
  fullName.value = 'Jane Doe';
};
</script>

<template>
  <h1>Vue 3 入門</h1>
  <h2>fullName: {{ fullName }}</h2>
  <button @click="changeName">Change Name</button>
</template>
```
ボタンをクリックするとデベロッパーツールのコンソールには”Write operation failed: computed value is readonly”が表示され書き込み処理が失敗、Computed プロパティは Readonly だというメッセージです。
- Computed プロパティの値は直接は更新できないことがわかりました。

## Setter を利用することで Compute プロパティを更新
Computed プロパティを使って更新を行いたい場合は直接ではなく Setter を利用する必要があります。
- 下記が Setter を利用して Computed プロパティを更新するためコードです。
```javascript
const fullName = computed({
  get() {
    return `${user.firstName} ${user.lastName}`;
  },
  set(newValue) {
    const names = newValue.split(' ');
    user.firstName = names[0];
    user.lastName = names[names.length - 1];
  },
});
```
get メソッドは Computed プロパティにアクセスして表示する場合に実行されるメソッドなので処理の内容は先ほどの Computed プロパティでの設定と同じです。
- set メソッドは新しい値 newValue を受け取り reactive な変数 user の firstName と lastName に受け取った newValue(ここでは Jane Doe)を空白で分割してそれぞれに設定しています。

changeName ボタンをクリックすると Computed プロパティを経由して更新が行われ”John Doe”から Jane Doe”となります。

!!! info Setter を利用することで Compute プロパティが更新できることがわかりました。

