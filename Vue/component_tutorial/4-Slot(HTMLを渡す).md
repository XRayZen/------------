- [Slot](#slot)
- [Slot のシンプルな例](#slot-のシンプルな例)
  - [props と Slotの違い](#props-と-slotの違い)
- [初期値の設定](#初期値の設定)
- [複数の Slots の設定](#複数の-slots-の設定)
- [Scoped Slot の設定](#scoped-slot-の設定)
- [少し複雑な Scoped Slot 設定](#少し複雑な-scoped-slot-設定)

# Slot
親から子コンポーネントに値を渡す場合に Props を利用することができましたが HTML などのコンテンツを渡したい場合には Slot を利用することができます。

# Slot のシンプルな例
components フォルダに User.vue ファイルを作成します。
- 親コンポーネントから受け取ったコンテンツを表示したい場所に slot タグを設定します。
```html
<template>
  <p>このユーザの名前は<slot></slot>です。</p>
</template>
```
App.vue ファイルでは User コンポーネントを import して User タプの間にコンテンツを挿入します。
```html
<script setup>
  import User from './components/User.vue';
</script>

<template>
  <h1>Vue 3 入門</h1>
  <User>John Doe</User>
</template>
```
User タグの間に挿入した”John Doe”が User コンポーネントの template タグで設定した slot の場所に表示されていることが確認できます。

このように Slot を利用することができます。

このようにコンテンツが文字列のようなシンプルな場合は props でも対応することができます。
```html
<script setup>
  const props = defineProps(['name']);
</script>
<template>
  <p>このユーザの名前は{{ props.name }}です。</p>
</template>
<template>
  <h1>Vue 3 入門</h1>
  <User name="John Doe" />
</template>
```
## props と Slotの違い
props との違いも明確にするため Slot を利用することで文字列ではなく HTML も設定できることを確認しておきます。
- User コンポーネントは最初の例のものを利用します。
```html
<template>
  <p>このユーザの名前は<slot></slot>です。</p>
</template>
```
今回は User タグの間に span タグに style 属性が設定されたコンテンツを挿入します。
```html
<script setup>
  import User from './components/User.vue';
</script>

<template>
  <h1>Vue 3 入門</h1>
  <User>
    <span style="font-weight: 900; font-size: 1.4em">John Doe</span>
  </User>
</template>
```
style 属性が適用された”John Doe”を確認することができます。

!!! error span のコンテンツを props に設定するとそのまま html タグが表示されます。

# 初期値の設定
slot タグに初期値を設定したい場合は slot タグの間にコンテンツを挿入しておくことで初期値として表示されます。
- User コンポーネントの slot タグの間に span タグで構成されたコンテンツを挿入しています。
```html
<template>
  <p>
    このユーザの名前は<slot
      ><span style="font-weight: 900; font-size: 1.4em; color: red"
        >John Doe</span
      ></slot
    >です。
  </p>
</template>
```
親コンポーネントでは User タグの中には何も挿入しません。
```html
<template>
  <h1>Vue 3 入門</h1>
  <User />
</template>
```
slot タグの間に挿入したコンテンツが初期値として利用されます。

!!! info slot に初期値を設定した場合でも親コンポーネントでタグの間にコンテンツを挿入した場合は親コンポーネントでの設定が優先され表示されます。
    ```html
    <template>
      <h1>Vue 3 入門</h1>
      <User><span style="color: blue">Jane Doe</span></User>
    </template>
    ```

# 複数の Slots の設定
User コンポーネントでは slot の設定場所は 1 箇所でしたが複数箇所設定することもできます。
- 複数 Slot を設定した場合の設定方法について確認していきます。

!!! info slot を複数設定する場合はそれぞれの slot を識別するための ID である名前が必要となります。
    そのため Named Slot(名前付き Slot)と呼ばれます。
    - 名前をつける場合は slot タグの name 属性にコンポーネント内で一意となる名前をつけます。
    - この名前は親コンポーネントからどの場所にコンテンツを挿入するか指定する際に利用されます。
```html
<template>
  <div>
    <slot name="title"></slot>
  </div>
  <div>
    <slot name="content"></slot>
  </div>
  <div>
    <slot name="actions"></slot>
  </div>
</template>
```
Named Slot にコンテンツを渡す親コンポーネントでは template タグに v-slot を設定します。v-slot には User.vue コンポーネントでつけた name を指定します。
```html
<template>
  <h1>Vue 3 入門</h1>
  <User>
    <template v-slot:title><h1>ユーザ情報</h1></template>
    <template v-slot:content
      ><div>
        <div>John Doe</div>
        <div>Jane Doe</div>
      </div></template
    >
    <template v-slot:actions><button>ユーザ追加</button></template>
  </User>
</template>
```
v-slot に設定した name の場所にそれぞれで設定したコンテンツが表示されます。

Named Slot を利用した場合でも明示的に name 属性を設定しない slot タグが default となります。
- name 属性を設定しなくてもコンテンツを受け取ることができます。
- 真ん中の slot から name=“content”を削除します。
```html
<template>
  <div>
    <slot name="title"></slot>
  </div>
  <div>
    <slot></slot>
  </div>
  <div>
    <slot name="actions"></slot>
  </div>
</template>
```
デフォルトの slot にコンテンツを渡す場合は template タグの v-slot を利用する必要がなく User タグの中に挿入したコンテンツがそのままデフォルトの slot に設定されます。

!!! warning v-slot がない template タグで囲んでもデフォルトの slot には設定されません。
```html
<template>
  <h1>Vue 3 入門</h1>
  <User>
    <template v-slot:title><h1>ユーザ情報</h1></template>
    <div>
      <div>John Doe</div>
      <div>Jane Doe</div>
    </div>
    <template v-slot:actions><button>ユーザ追加</button></template>
  </User>
</template>
```
もしデフォルトの slot を明示的に v-slot を使って設定したい場合には default を設定します。
```html
<template v-slot:default>
  <div>
    <div>John Doe</div>
    <div>Jane Doe</div>
  </div></template
>
```
Named Slot の場合も User コンポーネントの slot タグの間にコンテンツを挿入することでデフォルト値として設定することができます。
```html
<template>
  <div>
    <slot name="title">ユーザ</slot>
  </div>
  <div><slot>コンテンツ</slot></div>

  <div>
    <slot name="actions">アクション</slot>
  </div>
</template>
```
App コンポーネントの User タグでコンテンツを挿入しない場合はデフォルト値が表示されます。
```html
<template>
  <h1>Vue 3 入門</h1>
  <User />
</template>
```
# Scoped Slot の設定
Scoped Slot は slot を利用して子コンポーネント側からデータを渡し親コンポーネントからそのデータにアクセスを行うことができます。
- この機能については動作確認したほうがわかりやすいのシンプルな例を使って説明します。

子コンポーネント側で slot タグの中に props と同様に任意の名前の属性を追加し値を設定します。
- ここでは message という名前をつけて値にユーザを設定しています。

!!! INFO この message 属性のことを slot props と呼びます。
```html
<template>
  <slot message="ユーザ"></slot>
</template>
```
親コンポーネントでは slot props の値にアクセスするために template タグを利用します。

default の slot から slot props を受け取るので v-slotに任意の名前を設定します。
- ここでは slotProps という名前をつけています。

slotPops の中に User コンポーネントで設定した slot props である message が含まれているか確認するため slotProps をマスタッシュで囲みます。
```html
<template>
  <h1>Vue 3 入門</h1>
  <User>
    <template v-slot:default="slotProps"> {{ slotProps }} </template>
  </User>
</template>
```
ブラウザで確認すると slot props の中身を確認することができ子コンポーネントの slot タグで設定した message とその値を確認することができます。

親コンポーネントでは v-slot:default を設定したタグの中で slotProps にアクセスを行い message に設定した値を表示することができます。
- ブラウザには”ユーザ”が表示されます。
```html
<template>
  <h1>Vue 3 入門</h1>
  <User>
    <template v-slot:default="slotProps"> {{ slotProps.message }} </template>
  </User>
</template>
```
Named Slot による複数の slot の設定と 1 つの slot に複数の slot props が設定できるかの確認を行います。
- name 属性に header を設定した slot を追加し message という名前の slot props を追加し、default の slot には message の他、content の 2 つの slot props を設定しています。

!!! warning name 属性は slot を識別するための ID なので slot props ではないことに注意してください。
```html
<template>
  <slot name="header" message="ヘッダー"></slot>
  <slot message="ユーザ" content="コンテント"></slot>
</template>
```
App.vue ファイルで 2 つの template タグを使ってそれぞれの slot から slot props を取得しています。
```html
<template>
  <h1>Vue 3 入門</h1>
  <User>
    <template v-slot:header="slotProps">
      <div>{{ slotProps }}</div>
    </template>
    <template v-slot:default="slotProps">
      <div>{{ slotProps }}</div>
    </template>
  </User>
</template>
```
slot が複数の場合も slot props が複数ある場合も slot props が親コンポーネント側で取得できていることが確認できます。

slotProps に入っている props がわかっている場合は分割代入を利用することができます。
```html
<template>
  <h1>Vue 3 入門</h1>
  <User>
    <template v-slot:header="{ message }">
      <div>{{ message }}</div>
    </template>
    <template v-slot:default="{ message, content }">
      <div>{{ message }}/{{ content }}</div>
    </template>
  </User>
</template>
```
v-slot の省略系もあり v-slot を#に変更することもできます。
```html
<template>
  <h1>Vue 3 入門</h1>
  <User>
    <template #header="{ message }">
      <div>{{ message }}</div>
    </template>
    <template #default="{ message, content }">
      <div>{{ message }}/{{ content }}</div>
    </template>
  </User>
</template>
```
# 少し複雑な Scoped Slot 設定
シンプルな Scoped Slot の利用方法は理解できたと思うので少し難しい Scoped Slot の設定を行います。

子コンポーネント側で外部リソースから取得したデータを
- slot props を利用して親コンポーネントに渡し
- 親コンポーネントで表示の設定を行えることを確認します。

User コンポーネント上で fetch 関数を利用して JSONPlaceHolder を使ってユーザの一覧情報を取得します。
- JSONPlaceHolder は無料のサービスで`http://jsonplaceholder.typicode.com/users`にアクセスするとユーザの一覧が JSON で戻されます。

取得した users データは v-for ディレクティブを利用して slot タグの中で展開し slot props の user に設定を行い親コンポーネントに渡しています。
- 変数を渡す場合には v-bind ディレクティブを利用する必要があるため user に:(コロン)を設定します。
```html
<script setup>
  import { ref } from 'vue';

  const users = ref([]);

  const fetchPost = async () => {
    const res = await fetch('https://jsonplaceholder.typicode.com/users');
    users.value = await res.json();
  };
  fetchPost();
</script>
<template>
  <slot v-for="user in users" :user="user"></slot>
</template>
```
slot props の user を受け取った親コンポーネントでは user オブジェクトのプロパティを利用して自由に表示内容を設定することができます。
- 下記では name のみ利用しています。
```html
<template>
  <h1>Vue 3 入門</h1>
  <ul>
    <User>
      <template v-slot:default="{ user }">
        <li>{{ user.name }}</li>
      </template>
    </User>
  </ul>
</template>
```
ブラウザで確認するとユーザ名の一覧が表示されます。

親側で表示内容を変更できることによりあるコンポーネントで User コンポーネントを利用して namt と email だけを表示させたり、別のコンポーネントでは User コンポーネントを利用して name だけ表示させるといったことが可能になります。

!!! warning User コンポーネント側で表示の設定を行なっていた場合にはそれを利用するコンポーネントはいつも同じフォーマットでしかユーザ情報を表示することができません。
    User コンポーネントをより汎用的に利用することができます。

slot props をデフォルトの slot から受け取る場合は default を省略することもできます。
```html
<template>
  <h1>Vue 3 入門</h1>
  <ul>
    <User>
      <template v-slot="{ user }">
        <li>{{ user.name }}</li>
      </template>
    </User>
  </ul>
</template>
```










