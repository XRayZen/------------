

# reactive を利用してデータ共有(状態管理)
コンポーネント間でデータを共有したい場合に Props, Emit 先ほど説明を行なった Provide/Inject を利用しなくても Vue3 では Pinia というライブラリ(Vuex も同じ)を利用することでアプリケーション全体でデータを共有することができます。
- しかし、それらの機能、ライブラリを利用しなくても reactive な関数で定義したオブジェクトを別ファイルに分けて管理することでコンポーネント間でデータを共有することができます。

>[Vue.js Piniaを使って状態管理(データの共有)を行ってみよう](https://reffect.co.jp/vue/vue-pinia/)

src フォルダに store フォルダを作成しその下に countStore.js ファイルを作成します。
- countStore.js ファイルでは reactive 関数を利用して count の初期値と addCount を設定します。
- 設定した counter は他のファイルでも利用できるように export を行う必要があります。
```javascript
import { reactive } from 'vue';

export const counter = reactive({
  count: 0,
  addCount() {
    this.count++;
  },
});
```
count と addCount を利用したいコンポーネントでは countStore を import します。ここでは CompB コンポーネントで import を行います。count は counter.count, addCount は counter.addCount でアクセスすることができます。
```html
<script setup>
  import { counter } from '../store/countStore';
</script>

<template>
  <h3>CompBコンポーネント</h3>
  <p>Count:{{ counter.count }}</p>
  <button @click="counter.addCount">+</button>
</template>
```
ブラウザで確認するとボタンをクリックするとカウントの数が増えていくことが確認できます。countStore で定義した count の値を import したコンポーネントから操作することができます。

さらに他のコンポーネントでも countStore.js を import することでデータ共有できているのか確認するために ComA コンポーネントでは count のみ表示させます。もちろん addCount 関数を CompA コンポーネントでも利用することができます。
```html
<script setup>
  import CompB from './CompB.vue';
  import { counter } from '../store/countStore';
</script>

<template>
  <h2>CompAコンポーネント</h2>
  <p>Count:{{ counter.count }}</p>
  <CompB />
</template>
```
CompB コンポーネントの”+“ボタンをクリックすると CompA と CompB コンポーネントの count が増えることが確認できます。countStore.js ファイルで定義した reactive な変数をデータ共有として利用することができることがわかりました。

小さなプロジェクトであればデータ共有に利用することもできますが大きなプロジェクトの場合は Pinia を利用することになります。
- Pinia を利用した場合の Vue の Devtool を利用してデバッグを行うことができるため開発を効率的に進めることができます。


ここまで読み進めてもらえればコンポーネントにおける Props, emit, Slot, Provider/Inject の基礎はかなり理解できているのではないでしょうか。Vue アプリケーションを作成するためにはさらに Vue Router や状態管理の Vuex や Pinia などの学習が必要となります。


