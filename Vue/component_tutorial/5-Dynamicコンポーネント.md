- [Dynamic コンポーネント](#dynamic-コンポーネント)
  - [Dynamic コンポーネントの component タグと is 属性](#dynamic-コンポーネントの-component-タグと-is-属性)
  - [keep-alive](#keep-alive)


# Dynamic コンポーネント
Dynamic コンポーネントを利用することで表示するコンポーネントを動的に切り替えることができます。

ボタンを利用して画面に表示させる内容を切り替えることができる機能を実装するために components フォルダの中に 2 つのファイル Tokyo.vue, Kyoto.vue を作成します。

それぞれには下記の内容を記述します。
```html
<template>
  <h2>東京</h2>
  <p>日本の首都です。</p>
</template>
```
```html
<template>
  <h2>京都</h2>
  <p>日本の古都です。</p>
</template>
```
App.vue ファイル内で 2 つのファイルを import して下記のように記述します。
```html
<script setup>
  import Tokyo from './components/Tokyo.vue';
  import Kyoto from './components/Kyoto.vue';
</script>

<template>
  <h1>Vue 3 入門</h1>
  <div>
    <button>東京</button>
    <button>京都</button>
  </div>
  <div>
    <Tokyo />
    <Kyoto />
  </div>
</template>
```
切り替えの機能は実装していないので 2 つのボタンと 2 つのコンポーネントの内容がそのまま表示されます。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-40-1024x652.webp)

東京のボタンを押すと東京の内容、京都のボタンを押すと京都の内容が表示されるように reactive な変数と v-if ディレクティブと click イベントを利用します。
```html
<script setup>
  import { ref } from 'vue';
  import Tokyo from './components/Tokyo.vue';
  import Kyoto from './components/Kyoto.vue';

  const city = ref('tokyo');
</script>

<template>
  <h1>Vue 3 入門</h1>
  <div>
    <button @click="city = 'tokyo'">東京</button>
    <button @click="city = 'kyoto'">京都</button>
  </div>
  <div>
    <Tokyo v-if="city == 'tokyo'" />
    <Kyoto v-else />
  </div>
</template>
```
設定により東京ボタンをクリックすると東京の内容が表示され、京都のボタンを押すと京都の内容が表示されます。
- v-if ディレクティブを利用して表示・非表示を切り替えを行なっていますがこの機能は Dynamic コンポーネントの機能を利用することで変更することができます。

## Dynamic コンポーネントの component タグと is 属性
Dynamic コンポーネントでは component タグと is 属性を利用します。
- is 属性は v-bind により変数を設定することができ、is 属性の値を変更することで動的にコンポーネントを設定できるようになります。

script setup を利用した場合については component タグと is 属性を利用して下記のように設定を行うことで Dynamic コンポーネントを利用することができます。
```html
<script setup>
  import { ref, computed } from 'vue';
  import Tokyo from './components/Tokyo.vue';
  import Kyoto from './components/Kyoto.vue';

  const city = ref('tokyo');

  const tabs = {
    tokyo: Tokyo,
    kyoto: Kyoto,
  };
  const tab = computed(() => tabs[city.value]);
</script>

<template>
  <h1>Vue 3 入門</h1>
  <div>
    <button @click="city = 'tokyo'">東京</button>
    <button @click="city = 'kyoto'">京都</button>
  </div>
  <component v-bind:is="tab"></component>
</template>
```
tabs と computed プロパティを利用することで is 属性に設定する tab の情報にはコンポーネントの名前ではなくコンポーネントの情報が含まれています。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-41-1024x230.webp)

変数の city の値をそのまま is の値として設定した方法ではコンポーネントの内容がブラウザ上に表示されることはありません。
- デベロッパーツールで要素を見ると tokyo の場合はタグが表示されています。
```html
<template>
  <h1>Vue 3 入門</h1>
  <div>
    <button @click="city = 'tokyo'">東京</button>
    <button @click="city = 'kyoto'">京都</button>
  </div>
  <keep-alive> <component v-bind:is="city"></component></keep-alive>
</template>
```
## keep-alive
Dynamic コンポーネントを利用することでボタンをクリックすることでコンポーネントを切り替えることができるようになりました。

しかし is 属性を利用してコンポーネントを切り替えた場合にコンポーネントの中の状態を保持することができないため各コンポーネントで状態を保持したい場合は keep-alive タグで囲むことで状態を保持することができます。
- 状態を保持するとはどういうことかということを含めて動作確認を行います。

Tokyo コンポーネントにカウンターを追加します。
- “Add count”をクリックすると count 数が増えるだけの機能です。
```html
<script setup>
  import { ref } from 'vue';

  const count = ref(0);
</script>

<template>
  <h2>東京</h2>
  <p>日本の首都です。</p>
  <p>{{ count }}</p>
  <button @click="count++">Add Count</button>
</template>
```
デフォルトでは count の値は 0 なのでブラウザ上には 0 が表示されます。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-43-1024x633.webp)

ボタンをクリックすると count 数が増えます。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-42-1024x633.webp)

次に”京都”ボタンをクリックして、再度”東京”ボタンを押してください。状態を保持できないために表示されていた count が 0 になります。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-43-1024x633.webp)

count 数を保持するために component タグを keep-alive タグで囲みます。
- これで設定は完了です。
```html
<template>
  <h1>Vue 3 入門</h1>
  <div>
    <button @click="city = 'tokyo'">東京</button>
    <button @click="city = 'kyoto'">京都</button>
  </div>
  <keep-alive> <component v-bind:is="tab"></component></keep-alive>
</template>
```
再度同じことを実施すると count 数がタブを切り替えても count の値が保持できることが確認できます。
- input 要素などの入力フォームで入力した値を保持したい場合も keep-alive を利用することができます。



