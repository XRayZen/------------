- [Provide/Inject](#provideinject)
- [props を利用した場合](#props-を利用した場合)
- [Provide/Inject を利用した場合](#provideinject-を利用した場合)
- [Reactive な変数を渡す](#reactive-な変数を渡す)
- [子コンポーネントで更新(関数を渡す)](#子コンポーネントで更新関数を渡す)


# Provide/Inject
親コンポーネントから子コンポーネントにデータを渡したい時には props を利用することができます。

!!! error しかし親コンポーネントからデータを渡したい子コンポーネントまでの階層が深い場合は props を利用することは非常に手間がかかります。

!!! success props の代わりに Provide/Inject を利用することで親コンポーネントからあるコンポーネントに props を利用することなくデータを渡すことができます。

図で表すと Provide/Inject と props の違いは下記のように記述できます。
- props を利用して App コンポーネントから CompB コンポーネントにデータを渡す際は CompA を経由させる必要があります。
- しかし Provide/Inject を利用すると App コンポーネントから CompA を介さず CompB に直接データを渡すことができます。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue3_provide_inject.webp)

動作確認を行うために components フォルダに CompA.vue, CompB.vue ファイルを作成します。
- CompA.vue ファイルの中で CompB コンポーネントを import しています。
```html
<script setup>
  import CompB from './CompB.vue';
</script>
<template>
  <h2>CompAコンポーネント</h2>
  <CompB />
</template>
```
```html
<template>
  <h3>CompBコンポーネント</h3>
</template>
```
App.vue ファイルの中で CompA コンポーネントを import しています。
```html
<script setup>
  import CompA from './components/CompA.vue';
</script>

<template>
  <h1>Vue 3 入門</h1>
  <CompA />
</template>
```
ブラウザ上には以下の画面が表示されます。
>階層のあるコンポーネントを表示
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-44-1024x615.webp)

# props を利用した場合
Provider/Inject の動作確認する前に App コンポーネントから CompB コンポーネントに props を利用してデータを渡す方法を復習しておきましょう。
- message という名前の props に”props でデータ渡し”を設定して CompA に渡します。
```html
<template>
  <h1>Vue 3 入門</h1>
  <CompA message="propsでデータ渡し" />
</template>
```
CompA では受け取った props の message を CompB に渡します。
- この場合は message に v-bind を設定して渡します。
```html
<script setup>
  import CompB from './CompB.vue';

  const { message } = defineProps(['message']);
</script>
<template>
  <h2>CompAコンポーネント</h2>
  <CompB :message="message" />
</template>
```
CompB では受け取った props の message を template タグで表示させています。
```html
<script setup>
  const { message } = defineProps(['message']);
</script>

<template>
  <h3>CompBコンポーネント</h3>
  <p>{{ message }}</p>
</template>
```
ブラウザ上には props を介して渡されたデータ表示されます。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-45-1024x635.webp)
これが props を利用した方法です。

# Provide/Inject を利用した場合
props でのデータの渡し方が確認できたので Provide/Inject を利用して App から CompB にデータを渡します。
- Provide と Inject を利用するためには vue から provide 関数と inject 関数を import する必要があります。

App.vue ファイルでは provider 関数を利用して message に文字列”Provide/Inject でデータ渡し”を設定します。
- 第一引数には key, 第二引数には value を設定しています。

props とは異なり親コンポーネントの template タグで何か設定を行うことはありません。
```html
<script setup>
  import CompA from './components/CompA.vue';
  import { provide } from 'vue';

  provide('message', 'Provide/Injectでデータ渡し');
</script>

<template>
  <h1>Vue 3 入門</h1>
  <CompA message="propsでデータ渡し" />
</template>
```
CompB コンポーネントで Inject 関数を利用して Provide で設定した値を key を利用して取得します。
- これで App から ComB にデータを渡すことができました。
```html
<script setup>
  import { inject } from 'vue';
  const { message } = defineProps(['message']);

  const message2 = inject('message');
</script>

<template>
  <h3>CompBコンポーネント</h3>
  <p>{{ message }}</p>
  <p>{{ message2 }}</p>
</template>
```
ブラウザで確認すると App の provide 関数で指定した文字列が表示されます。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-46-1024x635.webp)

Provide/Inject の設定方法だけではなく props の方法と違いが理解できたか
- scriptタグ内で簡単に渡せた

# Reactive な変数を渡す
文字列ではなく reactive な変数を渡した場合の動作確認も行います。
- ref 関数で message を定義して初期値に同じ文字列を設定します。
```html
<script setup>
  import CompA from './components/CompA.vue';
  import { provide, ref } from 'vue';
  const message = ref('Provide/Injectでデータ渡し');

  provide('message', message);
</script>
```
CompB での設定変更は必要ありません。
- ブラウザを確認すると先ほどと同じ結果となります。

reactive な変数なので親コンポーネントで iput 要素を利用して更新してもその更新が Provide / Inject を経由して CompB に反映されるのか確認します。
```html
<script setup>
  import CompA from './components/CompA.vue';
  import { provide, ref } from 'vue';
  const message = ref('Provide/Injectでデータ渡し');

  provide('message', message);
</script>
```
input 要素で文字列の更新を行うとその更新が反映されることが確認できます。

# 子コンポーネントで更新(関数を渡す)
Inject で渡された値を直接子コンポーネントで更新することができません。
- Provide/Inject では関数も渡すことができるのでその関数を利用して子コンポーネントから更新を行うことができます。

App で reactive な変数 count を定義して、関数で更新できるように addCount を追加します。
- count と同様に追加した addCount も provide 関数に設定することができます。
```html
<script setup>
import CompA from './components/CompA.vue';
import { provide, ref } from 'vue';

const count = ref(0);

const addCount = () => {
  count.value++;
};

provide('count', count);
provide('addCount', addCount);
</script>
```
count, addCount を inject 関数で取得します。
```html
<script setup>
  import { inject } from 'vue';
  const { message } = defineProps(['message']);

  const count = inject('count');
  const addCount = inject('addCount');
</script>

<template>
  <h3>CompBコンポーネント</h3>
  <p>{{ message }}</p>
  <p>Count:{{ count }}</p>
  <button @click="addCount">+</button>
</template>
```
ボタンをクリックすると Count の数がクリック毎に増えていくことが確認できます。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-48-1024x635.webp)

Provide 関数で変数 count と関数 addCount を分けて設定しましたがオブジェクトで 1 つにまとめて設定を行うこともできます。
```html
<script setup>
  import CompA from './components/CompA.vue';
  import { provide, ref } from 'vue';

  const count = ref();

  const addCount = () => {
    count.value++;
  };

  provide('count', {
    count,
    addCount,
  });
</script>
```
ComB コンポーネントでは inject を利用して count のオブジェクトを受け取り分割代入で count, addCount を取り出して利用します。
- ブラウザ上での動作は先ほどと変わりません。
```html
<script setup>
  import { inject } from 'vue';
  const { message } = defineProps(['message']);

  const { count, addCount } = inject('count');
</script>
```








