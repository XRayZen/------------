
# Watcher
watcher を利用することで reactive な変数、computed プロパティの変更を監視し、変更を検知した場合に別の処理を実行することができます。

## watcher + ref
ref 関数で作成した reactive な変数 count を監視して count が更新されたら更新された後の値を表示するように設定を行います。
```html
<script setup>
import { ref, watch } from 'vue';

const count = ref(0);

watch(count, (count) => {
  console.log('count:', count);
});
</script>

<template>
  <h1>Vue 3 入門</h1>
  <button @click="count++">Count:{{ count }}</button>
</template>
```
Count のボタンを押す度に watcher で変更を検知し、デベロッパーツールのコンソールに count 数が表示されます。

!!! note watcher では変更後の値だけではなく変更前の値も確認することができます。
    第一引数に変更後の値、第二引数に変更前の値が渡されます。
    ```javascript
    watch(count, (count, previousCount) => {
      console.log('count:', count);
      console.log('previousCount:', previousCount);
    });
    ```
    Count ボタンをクリックすると変更前の値と変更後の値がコンソールに表示されます。

## watcher + reactive
reactive 関数を使って定義した count に対しても watcher の動作確認を行っておきます。監視対応に state.count を設定します。
```html
<script setup>
  import { reactive, watch } from 'vue';

  const state = reactive({
    count: 0,
  });

  watch(state.count, (count, previousCount) => {
    console.log('count:', count);
    console.log('previousCount:', previousCount);
  });
</script>

<template>
  <h1>Vue 3 入門</h1>
  <button @click="state.count++">Count:{{ state.count }}</button>
</template>
```
!!! error ページを開くと下記のメッセージがデベロッパーツールのコンソールに表示され watch のデータソースとして state.count が利用できないことがわかります。
```text
[Vue warn]: Invalid watch source:  0 A watch source can only be a getter/effect function, a ref, a reactive object, or an array of these types
```
!!! success reactive 関数のオブジェクトのプロパティを監視する場合は下記の関数の形に変更することで state.count の監視が可能になります。
    Count ボタンをクリックすると変更前の値と変更後の値がコンソールに表示されます。
```javascript
watch(
  () => state.count,
  (count, previousCount) => {
    console.log('count:', count);
    console.log('previousCount:', previousCount);
  }
);
```

# watcher の Options
reactive 関数で定義した state を watch に設定して count を変更した場合の動作も確認しておきます。
## 中のオブジェクトの変更を検知
- 下記の設定を行ってもコンソールには何も表示されません。
```javascript
watch(
  () => state,
  (state, previousState) => {
    console.log('state:', state);
    console.log('previousState:', previousState);
  }
);
```
state を watcher に設定して中のオブジェクトの変更を検知するために options の deep を true に変更する必要があります。
- deep の設定値を true に変更すると変更は検知することができますが count の変更前の値は取得することはできません。
```javascript
watch(
  () => state,
  (state, previousState) => {
    console.log('state:', state);
    console.log('previousState:', previousState);
  },
  { deep: true }
);
```
watch の第一引数に state をそのまま設定するとオプションの deep を true に設定していなくても count の変更を検知してくれます。
- この場合も count の変更前の値は取得することはできません。
```javascript
watch(
  state,
  (state, previousState) => {
    console.log('state:', state);
    console.log('previousState:', previousState);
  },
  { deep: true }
);
```
## watcher の処理をページを開いた直後に実行しておきたい
watcher の処理をページを開いた直後に実行しておきたい場合はオプションに immediate:true を設定することで可能です。
- デフォルトでは false が設定されています。
```javascript
watch(
  () => state.count,
  (count, previousCount) => {
    console.log('count:', count);
    console.log('previousCount:', previousCount);
  },
  { immediate: true }
);
```
初期値は 0 で previousCount の値はないのでコンソールには下記が表示されます。
```text
count: 0
previousCount: undefined
```
# watchEffect
reactive な変数と computed プロパティの変更の検知は watcher だけではなく watchEffect でも行うことができます。

watch のように特定の変数を指定するのではなく watchEffect の中に記述した関数で記述されている変数の変更を検知して実行されます。

下記のように ref 関数で count, count2 を定義します。watchEffect 関数の中では count のみ利用します。
- 2 つボタンを設定して一つは count の変更、もう一つは count2 の変更を行います。
- Count のボタンをクリックした場合には watchEffect の関数が実行され、Count2 をクリックしても何も行われません。
```html
<script setup>
  import { ref, watchEffect } from 'vue';

  const count = ref(0);
  const count2 = ref(100);

  watchEffect(() => console.log(count.value));
</script>

<template>
  <h1>Vue 3 入門</h1>
  <button @click="count++">Count:{{ count }}</button>
  <button @click="count2++">Count2:{{ count2 }}</button>
</template>
```
watchEffect ではページを開いた時にも一度処理が実行されます。

両方の変数を watchEffect に追加するとどちらのボタンをクリックしても watchEffect の関数が実行されるようになります。
```javascript
watchEffect(() => console.log(`${count.value}/${count2.value}`));
```
続きはコンポーネント編で


