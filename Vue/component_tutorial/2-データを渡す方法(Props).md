
- [データを渡す方法(Props)](#データを渡す方法props)
- [Props の設定方法](#props-の設定方法)
  - [型を設定せず配列で名前を指定することも可能](#型を設定せず配列で名前を指定することも可能)
- [複数の props を渡す場合](#複数の-props-を渡す場合)
  - [分割代入を利用](#分割代入を利用)
- [初期値の設定](#初期値の設定)
  - [props を必ず受け取りたい場合](#props-を必ず受け取りたい場合)
- [数値を渡したい場合](#数値を渡したい場合)
- [Boolean を渡したい場合](#boolean-を渡したい場合)
- [reactive な変数を渡したい場合](#reactive-な変数を渡したい場合)
- [reactive な変数を更新](#reactive-な変数を更新)
  - [渡された props を子コンポーネントで直接更新してはいけません。](#渡された-props-を子コンポーネントで直接更新してはいけません)
  - [親コンポーネントで定義されている props の値を更新したい場合](#親コンポーネントで定義されている-props-の値を更新したい場合)
- [class 属性の設定を渡したい場合](#class-属性の設定を渡したい場合)
  - [タグで設定した class 属性をそのまま子コンポーネントのタグに指定したい場合](#タグで設定した-class-属性をそのまま子コンポーネントのタグに指定したい場合)
  - [ルートの要素に class を適用させたくない場合](#ルートの要素に-class-を適用させたくない場合)
- [useAttrs による属性の取得](#useattrs-による属性の取得)
  - [useAttr の内容から style 属性を利用したい場合](#useattr-の内容から-style-属性を利用したい場合)


# データを渡す方法(Props)
ここまでの説明でコンポーネントとは完全に独立したコンポーネントでコンポーネントタグを template タグに組み込むと必ず同じ内容がブラウザ上に表示されることがわかりました。

コンポーネントは独立しており count のように内部で変数を定義して変数を更新することができましたがそれだけではなく外側からデータを受け取ることができ、データを受け取る際には Props(properits の略)を利用します。

![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue3_props.webp)

# Props の設定方法
Hello コンポーネントにデータを渡したい場合は Hello タグの中に任意の名前の属性名(message)設定し渡したい文字列”Props の使い方”を設定します。
- コンポーネント側でデータを受け取る時に設定した名前を利用するので何のデータを渡しているのかわかりやすい名前をつけます。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello message="Propsの使い方" />
</template>
```
データを受け取る側の Hello コンポーネントの内部では props を受け取るための設定が必要となります。

props を受け取る場合は defineProps 関数を利用しますが利用する際に import を行う必要はありません。
- defineProps の中で受け取る props の名前と型を設定します。
  - 型には文字列を表す String の他に Number, Boolean, Object, Array などを設定することができます。
- 渡される値の型によって設定を変更します。
  - 型を設定することでどのようなデータが渡されるかがわかることと異なる型でデータが渡された場合に警告メッセージとして表示されます。
- どのような警告メッセージが表示されるかは後ほど確認します。

defineProps の戻り値を保存する変数名を props としていますが好きな名前をつけることができます。
- props 変数の中にオブジェクトとして message が含まれています。
```html
<script setup>
  const props = defineProps({
    message: String,
  });
</script>

<template>
  <h2>初めてのコンポーネント</h2>
  <p>{{ props.message }}</p>
</template>
```
message に設定した”Props の使い方”を Hello コンポーネントで表示することができました

## 型を設定せず配列で名前を指定することも可能
props については型を設定せず配列で名前を指定することで利用することもできます。
```html
<script setup>
  const props = defineProps(['message']);
</script>

<template>
  <h2>初めてのコンポーネント</h2>
  <p>{{ props.message }}</p>
</template>
```
複数の Hello コンポーネントを利用する場合に message の値を変更することで異なる内容を表示することができます。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello message="Propsの使い方" />
  <Hello message="defineProps関数を利用" />
</template>
```
Hello コンポーネントを再利用していますが message に設定する値を外側から変更することで同じコンポーネントでも異なる内容を表示できるようになりました。

# 複数の props を渡す場合
複数の props を渡したい場合の設定方法も確認しておきます。
- 2 つの props を渡したい場合には名前の異なる属性名を 2 つ追加してそれぞれに値を設定します。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello message="Propsの使い方" name="Jonh" />
</template>
```
Hello コンポーネントでは defineProps に 2 つの props を設定する必要があります。
```html
<script setup>
const props = defineProps({
  message: String,
  name: String,
});
</script>

<template>
  <h2>初めてのコンポーネント</h2>
  <p>{{ props.name }} {{ props.message }}</p>
</template>
```
ブラウザで確認すると 2 つの props が表示されていることが確認できます。

## 分割代入を利用
defineProps から props 変数に戻されるデータはオブジェクトなので分割代入を利用することもできます。
- props の中から name, message を取り出しています。
- template タグでは”props.”を記述する必要がなくなります。
```html
<script setup>
  const { name, message } = defineProps({
    message: String,
    name: String,
  });
</script>

<template>
  <h2>初めてのコンポーネント</h2>
  <p>{{ name }} {{ message }}</p>
</template>
```

# 初期値の設定
Hello コンポーネントの defineProps で定義した props の一部を親コンポーネントから渡さない場合の動作確認を行います。
- Hello.vue ファイルでは defineProps で message, name を設定しています。
```html
<script setup>
const props = defineProps({
  message: String,
  name: String,
});
</script>
```
Hello コンポーネントでは definProps で 2 つの props を設定していますが App コンポーネントでは Hello タグに message 属性だけ設定しています。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello message="Propsの初期値の設定" />
</template>
```
上記の設定後ブラウザで確認するとエラーメッセージが出ることもなく message に設定した”Props の初期値の設定”のみ表示されます。

name を設定しなくてもエラーメッセージは出ませんでしたが props には初期値を設定することができます。
- 初期値が設定されている props が渡されなかった場合には初期値が利用されます。

初期値を設定する場合はオブジェクトの中に default プロパティを設定して初期値を設定します。
- props の設定をオブジェクトに変更した場合の型の設定は type プロパティで行います。

name のみ初期値の設定を行っています。
```html
<script setup>
const props = defineProps({
  message: String,
  name: { type: String, default: 'Jane' },
});
</script>
```
先ほどと同様に Hello コンポーネントには name を渡さずに設定を行います。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello message="Propsの初期値の設定" />
</template>
```
App コンポーネントから Hello タグに props の name を設定していませんが defineProps で設定したデフォルト値が表示されることが確認できます。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-9-1024x592.webp)

## props を必ず受け取りたい場合
defineProps に設定した props を設定しなくてもエラーがでませんでしたが場合によっては props を必ず受け取りたい場合もあります。
- その場合には props が必須かどうか指定できる required プロパティを利用することができます。

defineProps 関数の中の props の message の required プロパティを true を設定します。
```html
<script setup>
const props = defineProps({
  message: {
    type: String,
    required: true,
  },
  name: { type: String, default: 'Jane' },
});
</script>
```
下記の App コンポーネントでは Hello タグに props の設定を行っていません。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello />
</template>
```
ブラウザ上には name の初期値に設定した Jane が表示されますがコンソールログには下記の警告メッセージが表示されます。required を true に設定することで props の message が渡されていないことがわかります。

Terminal
```text
[Vue warn]: Missing required prop: "message"
  at <Hello>
  at <App>
```
デフォルトでは required を false になっているので true から false に設定すると警告メッセージが表示されなくなります。

# 数値を渡したい場合
文字列ではなく数値を渡したい場合の props の設定を確認します。
- 数値を受け取りたい場合は type に Number を設定することができます。

template タグの中では受け取った値に対して 100 を足すだけのシンプルなコードで数値が渡されているのかどうか確認します。
```html
<script setup>
const props = defineProps({
  price: Number,
});
</script>

<template>
  <h2>初めてのコンポーネント</h2>
  <p>{{ props.price + 100 }}</p>
</template>
```
Hello に対してこれまでと同じ方法で props の price 属性を利用して 1000 を渡します。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello price="1000" />
</template>
```
ブラウザを見ると 1100 ではなく 1000100 と文字列の連結が行われていることがわかります。
- ”+“は数値の場合は足し算を行いますが文字列の場合は文字列の結合を行います。

コンソールを確認すると警告メッセージが表示されていることがわかります。
```text
[Vue warn]: Invalid prop: type check failed for prop "price". Expected Number, got String with value "1000".
  at <Hello price="1000">
  at <App>
```

!!! ERROR メッセージの内容を見ると props の price は数値の値となるはずが文字列の 1000 が渡されているということがわかります。
- 文字列と同じ方法では数値を渡すことができないことがわかりました。

props で数値を渡したい場合はその値が数値であることを示すために v-bind の設定を行います。price に対して v-bind を設定します。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello v-bind:price="1000" />
</template>
```
v-bind:を price に設定後に再度確認すると足し算が行われ 1100 が表示され警告メッセージの表示も解消していることがわかります。
- v-bind は省略して設定することができます。

# Boolean を渡したい場合
同じ例として Boolean の場合も確認しておきます。
- isAdmin という props を定義して type を Boolean とします。
  - isAdmin の値によって v-if ディレクティブの分岐を利用して表示内容を変えます。
```html
<script setup>
const props = defineProps({
  isAdmin: Boolean,
});
</script>

<template>
  <h2>初めてのコンポーネント</h2>
  <p v-if="props.isAdmin">管理者です。</p>
  <p v-else>管理者ではありません。</p>
</template>
```
Hello コンポーネントに isAdmin で false を設定します。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello isAdmin="false" />
</template>
```
!!! error v-if ディレクティブの分岐により”管理者ではありません。“と表示されると思いますが”管理者です。“が表示され、警告メッセージがコンソールに表示されます。
    boolean の値が渡されるはずなのに String で”false”が渡されているといった内容のメッセージです。

!!! success 解決方法は数値の場合と同じで v-bind を設定することで false が Boolean として渡され”管理者ではありません。“が表示されます。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello v-bind:isAdmin="false" />
</template>
```
props を利用して数値、Boolean など期待通りに動作しない場合は v-bind が設定されているのか確認を行ってください。

# reactive な変数を渡したい場合
親コンポーネントで定義した reactive な変数を props を利用して子コンポーネントに渡す方法を確認していきます。

App.vue ファイルに ref 関数で変数 name を定義します。
- Hello タグに name 属性を追加した定義した name 変数を設定します。
```html
<script setup>
import Hello from './components/Hello.vue';
import { ref } from 'vue';

const name = ref('John');
</script>

<template>
  <h1>Vue 3 入門</h1>
  <Hello name="name" />
</template>
```
子コンポーネントの Hello.vue ファイルでは props の name は文字列なので型を String に設定しています。
```html
<script setup>
const props = defineProps({
  name: String,
});
</script>

<template>
  <h2>子コンポーネント</h2>
  <p>Hello {{ props.name }}</p>
</template>
```
ブラウザで確認すると name props に設定した文字列 name がそのまま表示されます。

!!! error ref 関数の初期値に設定した”John”が表示されません。

!!! success ただの name という文字列ではなく name という変数を渡すためにはここでも v-bind を利用する必要があります。
    name props の前に:(コロン)をつけます。v-bind:でも大丈夫です。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello :name="name" />
</template>
```
v-bind を設定して保存した瞬間にブラウザ上の name が ref 関数で設定した初期値”John”へと変わります。

!!! success reactive な変数を props で利用したい場合には v-bind を利用する必要があることがわかりました。

# reactive な変数を更新
reative な変数なので親コンポーネントで値を変更した場合に子コンポーネントで表示されている値が更新されるのかを確認します。

click イベントを持つボタンを追加し、ボタンをクリックすると changeName 関数により name の値が”John”から”Jane”に更新します。
```html
<script setup>
  import { ref } from 'vue';
  import Hello from './components/Hello.vue';

  const name = ref('John');
  const changeName = () => {
    name.value = 'Jane';
  };
</script>

<template>
  <h1>Vue 3 入門</h1>
  <Hello :name="name" />
  <button @click="changeName">Change Name</button>
</template>
```
最初は”John”と表示されていますがボタンをクリックすると”Jane”に変更されることが確認できます。reactive な変数を親コンポーネントで更新すると props で渡されたコンポーネント側でのその更新が反映されることがわかりました。

アプリケーションを構築していくと必ず props で渡された reactive な変数を子コンポーネント側で更新したいという場面に出くわします。

## 渡された props を子コンポーネントで直接更新してはいけません。
”渡された props を子コンポーネントで直接更新してはいけません”というルールを理解を深めるために子コンポーネントで props を更新できるか確認しておきましょう。
```html
<script setup>
  const props = defineProps({
    name: String,
  });

  const changeName = () => {
    props.name = 'Ken';
  };
</script>

<template>
  <h2>子コンポーネント</h2>
  <p>Hello {{ props.name }}</p>
  <button @click="changeName">Change Name</button>
</template>
```
ボタンをクリックするとコンソールには readonly なので更新が行えないと警告メッセージが表示されます。
>Terminal
```text
Set operation on key "name" failed: target is readonly. Proxy{name: 'John'}
```
警告メッセージが表示されるので誤って更新することはないのではないかと思いますが次は props で渡す値をオブジェクトに変更します。変数名を name から person に変更しています。
```html
<script setup>
  import { ref } from 'vue';
  import Hello from './components/Hello.vue';

  const person = ref({
    name: 'John',
  });
</script>

<template>
  <h1>Vue 3 入門</h1>
  <Hello :person="person" />
</template>
```
props で受け取る型を String から Object に変更しています。
```html
<script setup>
  const props = defineProps({
    person: Object,
  });

  const changeName = () => {
    props.person.name = 'Ken';
  };
</script>

<template>
  <h2>子コンポーネント</h2>
  <p>Hello {{ props.person.name }}</p>
  <button @click="changeName">Change Name</button>
</template>
```
ボタンをクリックすると警告メッセージが表示されることなく更新は完了します。

!!! warning 警告メッセージが表示されないので注意する必要があります。
## 親コンポーネントで定義されている props の値を更新したい場合
では子コンポーネントで行う何かしらのアクションを元に親コンポーネントで定義されている props の値を更新したい場合はどのように行えばよいのでしょうか。

!!! info 子コンポーネントで親コンポーネントで定義した reactive な変数を更新したい場合には emit イベントを利用します。
    emit イベントについては後ほど説明します。

# class 属性の設定を渡したい場合
コンポーネントでは props を利用せずに id 属性や class 属性などコンポーネントタグに設定した属性を渡すことができます。

!!! info この機能の名前は”fallthrough attribute” といいます。

Hello タグにに class 属性を使って active クラスを設定します。
```html
<template>
  <h1>Vue 3 入門</h1>
  <Hello class="active" />
</template>
```
!!! info これまでの動作確認の理解すると子コンポーネントである Hello で defineProps で受け取る class の設定を行うのではと思いますが class 属性は props とは別の扱いになるため設定を行う必要がありません。

Hello コンポーネントの template がどのように記述されているのか確認を行います。
```html
<template>
  <h2>子コンポーネント</h2>
</template>
```
h2 タグのみ含まれていると確認できたのでデベロッパーツールで要素を確認します。

子コンポーネントの templete タグ内で記述した h2 タグに Hello タグで設定した class の active クラスがそのまま設定されていることが確認できます。
- class 属性は何の追加設定もなく子コンポーネントに渡すことができることがわかりました。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-16-1024x232.webp)

子コンポーネントの h2 タグには class 属性を設定していませんでしたが class 属性が設定されている場合にはどのような動作になるのか確認します。
```html
<template>
  <h2 class="info">子コンポーネント</h2>
</template>
```
その場合は Hello タグで設定した class と h2 タグで設定した class がマージされて設定されます。
- どちらかが上書きされることがないこと、class 属性は props とは別の仕組みで子コンポーネントに渡されることがわかりました。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-17-1024x232.webp)

ここで子コンポーネントのルートの要素に 2 つの要素が存在する場合はどのなるのか疑問を持っている人もいるかもしれないので確認します。

2 つの要素が存在するとは template タグの直下に h2 タグと p タグが 2 つある状況です。
- 先ほどは h2 タグが 1 つだけ存在しました。
```html
<template>
  <h2 class="info">子コンポーネント</h2>
  <p>class属性の渡し方確認中</p>
</template>
```
デベロッパーツールで要素を確認します。active クラスがどこにも設定されていません。
- 子コンポーネントにおけるルートの要素が 2 つの場合にはコンポーネントで設定した class はどちらの要素にも設定されないということがわかりました。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-18-1024x232.webp)

その 2 つの要素を div で囲んだ場合はルートの要素が 1 つになり動作確認を行った h2 タグの状況と同じになるため class の active が設定されます。
```html
<template>
  <div>
    <h2 class="info">子コンポーネント</h2>
    <p>class属性の渡し方確認中</p>
  </div>
</template>
```
デベロッパーツールで確認すると div 要素に class=“active”が設定されていることがわかります。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-19-1024x232.webp)

## タグで設定した class 属性をそのまま子コンポーネントのタグに指定したい場合
!!! info Hello タグで設定した class 属性をそのまま子コンポーネントのタグに指定したい場合には`$attrs`を利用することができます。

pタグでclass属性にv-bindを設定し`$attrs.class`を設定します。
```html
<template>
  <div>
    <h2 class="info">子コンポーネント</h2>
    <p :class="$attrs.class">class属性の渡し方確認中</p>
  </div>
</template>
```
デベロッパーツールの p タグには class 属性が設定されていることがわかります。div 要素に設定された class はそのまま適用されています。
- `$attrs`の中には親コンポーネントから渡されたclasss属性の情報が含まれているので`$attrs` を利用することで子コンポーネントのルート要素の 1 箇所だけではなく何箇所にでも設定ができます。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-20-1024x232.webp)

## ルートの要素に class を適用させたくない場合
ルートの要素に class を適用させたくない場合は script タグを script setup タグとは別に追加し、inheritAttrs パラメータを false にすることで対応することができます。
- 下記のコードでは div タグへの class の適用はなくなり、`$attrs.class` を設定している p タグのみに class が適用されます。
```html
<script>
  export default {
    inheritAttrs: false,
  };
</script>
<script setup></script>

<template>
  <div>
    <h2 class="info">子コンポーネント</h2>
    <p :class="$attrs.class">style属性の渡し方確認中</p>
  </div>
</template>
```

# useAttrs による属性の取得
$attrsを利用することで子コンポーネントのタグに設定したclass属性の設定値を子コンポーネントで利用できることがわかりました。
- scriptタグないで$attrs に含まれる値を確認したい場合にはuseAttrs を利用することができます。
```html
<script setup>
  import {useAttrs} from 'vue'; const attrs = useAttrs(); console.log(attrs);
</script>
```
コンソールを見ると class に active が設定されていることが確認できます。
>Terminal
```text
Proxy{class: 'active', __vInternal: 1}
```
attrs を利用して class を適用したい場合は下記のように行うことができます。
```html
<p :class="attrs.class">class属性の渡し方確認中</p>
```
useAttrs を使って class 属性以外の id, style 属性を子コンポーネントに渡すことができるか確認します。
```html
<Hello id="main" style="color:red" class="active" />
```
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-21-1024x293.webp)
## useAttr の内容から style 属性を利用したい場合
useAttr の内容から style 属性を利用したい場合には下記のように行えることがわかります。
```html
<template>
  <h2 class="info">子コンポーネント</h2>
  <p :style="attrs.style">style属性の渡し方確認中</p>
</template>
```
!!! info $attrs.style で設定しても結果は同じです。

style タグでは colorを設定していたので attrs.style を設定した要素の文字は赤くなります。
![Alt text](https://reffect.co.jp/images/vue/vue-js-components/vue-3-components-22-1024x537.webp)

#　まとめ
props を利用せず id, class, style 属性を親コンポーネントから子コンポーネントに渡せること、利用方法を理解することができました。





