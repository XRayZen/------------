- [Reactivity](#reactivity)
- [ref の設定](#ref-の設定)
- [$ref の利用(Experimental Feature)](#ref-の利用experimental-feature)
- [reactive の設定](#reactive-の設定)
  - [script タグ内で count の数を増やす方法](#script-タグ内で-count-の数を増やす方法)
- [ref と reactive の違い](#ref-と-reactive-の違い)


# Reactivity
プロジェクト作成時からデフォルトで存在する Hello World コンポーネントでは画面に count ボタンが表示されておりボタンをクリックすると画面上の count の数が増えました。
- この機能の実装を行うためには Reactivity の理解が必要になります。
- ここでは Reactivity を行うとどうなるのかを確認しながら動作確認を行っていきます。

script タグで count を定義し click イベントを利用して count の数を増やせるか確認するためこれまでの確認した Vue.js の機能を利用して下記のコードを記述します。
```html
<script setup>
  const count = 0;
</script>

<template>
  <h1>Vue 3 入門</h1>
  <button type="button" @click="count++">count is: {{ count }}</button>
</template>
```
!!! error count 変数に設定した数字は表示されますが count ボタンをクリックしても count 数に変化はありません。
    この設定ではユーザの行動に対して何も変化が起こらないのでインタラクティブなアプリケーションではありません。

!!! success count 数が増えないのは定義した count 変数が reactivity を持っていないためです。
    ボタンをクリックすると画面上の count 数が増やせるようにするためには ref 関数または reactivity 関数を使って変数に reactivity を持たせる必要があります。

変数に reactivity を持たせる方法を確認していきます。

# ref の設定
ref 関数を利用することで変数に reactivity を持たせることができます。

vue から ref 関数を import して ref 関数の引数に count の初期値である 0 を設定します。
- たったこれだけの処理で count は reactive な変数となります。

ref 関数を利用して count を定義した後 count ボタンをクリックするとクリックした回数だけ count の数が増えます。
```html
<script setup>
  import { ref } from 'vue';
  const count = ref(0);
</script>

<template>
  <h1>Vue 3 入門</h1>
  <button type="button" @click="count++">count is: {{ count }}</button>
</template>
```
3 回ボタンをクリックすると count の数は 3 になります。

ref 関数で定義した reactive な変数 count はオブジェクトでラップされており value プロパティのみを持っています。
- そのため script タグ内では count = 1 といったように count を直接更新することはできません。
- script タグ内で count の値にアクセスする場合は count.value で行うことができます。
```html
<script setup>
  import { ref } from 'vue';
  const count = ref(0);
  console.log(count.value);
</script>
```
template タグ内ではでは count++でカウント数を増やすことができましたが script タグの中で count の数を増やす場合は下記のように count.value を利用する必要があります。
- count を増やすという処理は同じですが script タグと tempolate タグでは変更の方法が異なります。
```html
<script setup>
  import { ref } from 'vue';
  const count = ref(0);

  const addCount = () => {
    count.value++;
    console.log(count.value);
  };
</script>

<template>
  <h1>Vue 3 入門</h1>
  <button type="button" @click="addCount">count is: {{ count }}</button>
</template>
```
ref 関数を利用した reactive な変数の設定方法がわかりました。

# $ref の利用(Experimental Feature)
ref 関数を利用した場合に value のつけ忘れまたは value はつけるのが面倒だという人も多いと思います。
- Experimental Feature ですが script タグ内で value を利用しなくても reactive な変数として定義できる$ref があります。

$ref を利用するためには vite.config.js ファイルを設定する必要があります。
- plugins の vue の引数に reactivityTransform:true を追加しています。
```js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue({
      reactivityTransform: true,
    }),
  ],
});
```
設定を反映させるために npm run dev コマンドを再実行します。実行すると Experimental Feature なのでメッセージが表示されます。

メッセージが表示されますがそのまま$refを利用します。$ref を利用する場合は ref のように import を行う必要はありません。
```html
<script setup>
  const count = $ref(0);

  const addCount = () => {
    count++;
  };
</script>

<template>
  <h1>Vue 3 入門</h1>
  <button type="button" @click="addCount">count is: {{ count }}</button>
</template>
```
# reactive の設定
reactive な変数を定義する方法には ref 関数と reactive 関数があります。
- ref では引数には boolean, string, オブジェクトを取ることができますが reactive では引数にオブジェクトを取ります。
- ref と reactive では初期値の設定方法や値を更新する方法が異なります。

ref 関数と同様にボタンをクリックすると count の数が増える機能を reactive 関数を利用して行います。

reactive を利用する際も vue から reactive 関数を import する必要があります。
- reactive 関数では引数にオブジェクトを取るので count はオブジェクトのプロパティとして設定します。初期値は 0 とします。
```html
<script setup>
import { reactive } from 'vue';

const state = reactive({
  count: 0,
});

</script>
```
count の値を表示する場合はオブジェクトなので state.count となります。
- 値を増やす場合も state.count++となります。
```html
<template>
  <h1>Vue 3 入門</h1>
  <button type="button" @click="state.count++">
    count is: {{ state.count }}
  </button>
</template>
```
設定後にボタンをクリックすると count の数がクリックの度に増えてきます。

## script タグ内で count の数を増やす方法
template タグの中での count 数の増やし方はわかったので関数を利用して script タグ内で count の数を増やす方法を確認します。
- reactive 関数では ref 関数で作成した reactive な変数のように value プロパティを利用する必要はありません。
- 通常のオブジェクトの更新のように処理を行うことができます。
```html
<script setup>
  import { reactive } from 'vue';

  const state = reactive({
    count: 0,
  });

  const addCount = () => {
    state.count++;
  };
</script>

<template>
  <h1>Vue 3 入門</h1>
  <button type="button" @click="addCount">count is: {{ state.count }}</button>
</template>
```
ref 関数の引数にオブジェクトを取ることができると先ほど説明しました。
- ref 関数でのオブジェクトを利用した場合の動作も確認しておきましょう。
- オブジェクトの場合でも関数で count を更新する場合には value を介して行う必要があります。
- value を挟まなければ後の設定方法は reactive 関数と同じです。

template タグ内では state.count でアクセスすることができます。
```html
<script setup>
  import { ref } from 'vue';

  const state = ref({
    count: 0,
  });

  const addCount = () => {
    state.value.count++;
  };
</script>

<template>
  <h1>Vue 3 入門</h1>
  <button type="button" @click="addCount">count is: {{ state.count }}</button>
</template>
```

# ref と reactive の違い
ref 関数と reactive 関数の一番の違いは ref の場合は script タグ内で値にアクセスする場合に value を利用することです。
- reactive ではオブジェクトのみを扱うのに対して ref では確認したように primitive な値(string や boolean など)もオブジェクトも設定することができました。
