- [Binding(バインディング)](#bindingバインディング)
- [class 属性](#class-属性)
  - [style タグに設定した class を適用](#style-タグに設定した-class-を適用)
    - [v-bind を使って(動的に) class を設定する方法](#v-bind-を使って動的に-class-を設定する方法)
    - [通常の class とバインディング class を設定](#通常の-class-とバインディング-class-を設定)
    - [v-bind ディレクティブを設定した class の中に複数の class を設定](#v-bind-ディレクティブを設定した-class-の中に複数の-class-を設定)
    - [v-bind ディレクティブを設定した class の中では論理演算子(\&\&)を利用して設定](#v-bind-ディレクティブを設定した-class-の中では論理演算子を利用して設定)
- [style 属性の設定](#style-属性の設定)

# Binding(バインディング)
v-bind ディレクティブを利用することで html タグの属性の設定を行うことができます。
- 属性の設定を行うという意味もわかりにくいと思うので a タグで頻繁に利用される href 属性を使ってどういうものか確認していきます。

!!! info v-bind ディレクティブを利用して href 属性を設定する場合は下記のように記述することができます。
    v-bind の後ろの:(コロン)そして属性名が続きます。
    ```html
    <a v-bind:href="変数名">リンク名</a>
    ```

実際に href 属性に v-bind を設定する場合は以下のように行うことができます。
- 変数 link を定義して google の URL を設定しています。
```html
<script setup>
  const link = 'https://google.com';
</script>

<template>
  <h1>Vue 3 入門</h1>
  <div>
    <a v-bind:href="link">Google</a>
  </div>
</template>
```
v-bind によって href の値が link 変数に設定した google へのリンクになっているためアンダーバーのついた青色の文字列をクリックすると Google の検索画面が表示されます。

!!! question この動作を確認して href に変数が設定できて何かメリットがあるのか?
    href に変数が設定できるということはユーザの行う行動によって変数の値を変更することでユーザの行動に応じたリンク先を設定できるようになることを意味します。
    - ボタンをクリックすると Google のリンクから Yahoo へのリンクへと変更することもできます。
      - 静的な HTML だけで構成されたページではできないことです。
    - Vue.js を利用しなくても JavaScript で動的に変更できますが Vue.js を使うことで簡単にできるようになります。

!!! tip v-bind ディレクティブは省略形で記述することが可能で v-bind を削除して:変数名(変数名の前にコロンのみつける)でも設定が可能です。
    v-bind に限らず一般的に省略形がある場合は省略形をみんな利用するため v-bind:よりも:(コロン)のみの形をよく見かけます。

href 属性に設定を行いましたが他の属性でも利用することができます。

!!! example v-bind を頻繁に利用する例として以下のようなものがあります。
    動的に表示させる画像を変更したい場合、条件によってボタンを押させないといったことが v-bind ディレクティブを利用することで実装できます。
    - img タグの src 属性
    - button タグの disabled 属性

# class 属性
v-bind を利用して class 属性の設定を確認していきます。
- ここからはすべて v-bind:の省略型の:(コロン)を利用します。

!!! info Vue.js ではコンポーネントファイルの中に style タグがあるのでその中に class を設定することができます。
    style タグの中で active クラスを設定します。
    - active クラスでは適用すると文字が太文字で赤に設定されます。

## style タグに設定した class を適用
```html
<script setup></script>

<template>
  <h1>Vue 3 入門</h1>
  <p class="active">v-bindの設定方法の確認</p>
</template>

<style>
  .active {
    color: red;
    font-weight: 900;
  }
</style>
```
style タグの中に追加した active クラスを p タグに設定します。
- ブラウザで確認すると active クラスが反映されていることがわかります。
- style タグに class を追加することで template の html タグに style タグで定義した class が適用できることがわかりました。

### v-bind を使って(動的に) class を設定する方法
v-bind を使って class を設定する方法を確認していきます。
- class を設定するのであれば style タグに class を追加し、追加した class を class 属性に追加すればいいだけです。
- v-bind を利用して class を設定する理由は動的に適用する class を変更できるようにするためです。
  - その部分に注目して読み進めてみてください。

v-bind を利用した class の設定には複数の方法があるので状況に応じて使い分けてください。

最初の方法はオブジェクトを利用します。
- オブジェクトのプロパティに class 名を設定し、値に変数名または true, false を利用することで class を適用するかどうかを設定することができます。
- 変数名を true に設定した場合は class が適用され false に設定した場合は適用されません。

!!! question ここでなぜ true, false を設定しなければならないのか？
    v-bind を利用して class を設定する理由は動的に適用する class を変更できるようにするためだから

```javascript
v-bind:class="{class名: 変数名 or (true or false)}"
```
isActive という名前の変数を定義して false に設定します。
```html
<script setup>
  const isActive = false;
</script>

<template>
  <h1>Vue 3 入門</h1>
  <p>:class="{ active: isActive }">v-bindの設定方法の確認</p>
</template>
```
isActive の false の場合は active クラスが適用されないので class が適用されていない状態で表示されます。

isActive を true に設定することで active クラスが適用されます。
```html
<script setup>
  const isActive = true;
</script>
```
### 通常の class とバインディング class を設定
v-bind で設定する class 以外の class を設定したい場合(動的に変更する必要がなく常時設定させておきたい class)は通常の class 属性を利用することができます。
- class 属性に underLine クラスを設定します。
- p タグには class 属性と v-bind を設定した class が含まれることになります。
```html
<template>
  <h1>Vue 3 入門</h1>
  <p class="underLine" :class="{ active: isActive }">v-bindの設定方法の確認</p>
</template>

<style>
  .active {
    color: red;
    font-weight: 900;
  }
  .underLine {
    text-decoration: underline;
  }
</style>
```
変数 isActive の値が true の場合は active クラスと underline クラスが適用されることになります。
- underLine と active クラスを isActive が true の場合に適用したい場合には下記のように記述することができます。
```html
<p :class="{ 'underLine active': isActive }">v-bindの設定方法の確認</p>
```
### v-bind ディレクティブを設定した class の中に複数の class を設定
v-bind ディレクティブを設定した class の中には複数の class を設定することができます。
- back クラスを style タグの中に追加し、v-bind に利用する変数 isBlack を定義します。
  - isBlack の値は class が適用できるように true にしています。
- 2 つの変数を使って class を制御できることになったことで 4 つのパターンで p タグを表示できるようになりました。

!!! tip 下記のように複数の class を設定することもできます。
    ```html
    <p :class="{ 'underLine active': isActive, back: isBlack }"></p>
    ```
```html
<script setup>
  const isActive = true;
  const isBlack = true;
</script>

<template>
  <h1>Vue 3 入門</h1>
  <p class="underLine" :class="{ active: isActive, back: isBlack }">
    v-bindの設定方法の確認
  </p>
</template>

<style>
  .active {
    color: red;
    font-weight: 900;
  }
  .underLine {
    text-decoration: underline;
  }
  .back {
    background-color: black;
  }
</style>
```
どちらの変数も true の場合は 3 つの class が適用されているのでブラウザの画面は下記のように表示されます。
![Alt text](https://reffect.co.jp/images/vue/beginner-vue/vue3-beginner-14-1024x572.webp)

### v-bind ディレクティブを設定した class の中では論理演算子(&&)を利用して設定
v-bind ディレクティブを設定した class の中では論理演算子(&&)を利用して設定することもできます。
```html
v-bind:class="{変数名　&& 'class名'}"
```
isActive が true の時に active クラスが適用されます。
```html
<p :class="isActive && 'active'">v-bindの設定方法の確認</p>
```
利用頻度は低いとは思いますが論理演算子(||)も利用できます。
- 変数が false の時に class が適用されます。
```html
v-bind:class="{変数名　|| 'class名'}"
```
三項演算子を利用することもできます。
- isActive が true の場合は active, false の場合は underLine が適用されます。
```html
<p :class="isActive ? 'active' : 'underLine'">v-bindの設定方法の確認</p>
```
このように変数名を設定して変数名に設定している class を適用することもできます。
```html
<script setup>
  const isActive = 'active';
</script>

<template>
  <h1>Vue 3 入門</h1>
  <p>:class="isActive">v-bindの設定方法の確認</p>
</template>
```
複数の class を適用したい場合には配列を利用します。
```html
<script setup>
  const isActive = 'active';
  const isBlack = 'back';
</script>

<template>
  <h1>Vue 3 入門</h1>
  <p :class="[isActive, isBlack]">v-bindの設定方法の確認</p>
</template>
```
配列の中でオブジェクトを利用することもできます。
```html
<p :class="[{ active: 'isActive' }, isBlack]">v-bindの設定方法の確認</p>
```
配列の中で演算子を利用することもできます。
```html
<p :class="[isActive && 'active', isBlack]">v-bindの設定方法の確認</p>
<p :class="[isActive ? 'active' : 'underLine', isBlack]">
  v-bindの設定方法の確認
</p>
```
このように class を動的に変更するための方法がいくつかあります。

!!! info すべてを覚える必要はありませんが動的に class を変更する際にどの記述方法がアプリケーションに適しているかを考えて利用してみてください。

# style 属性の設定
style 属性の場合も class 同様に複数の記述方法があります。
- inline でオブジェクトを利用して複数のプロパティを設定することができます。
- 変数 activeColor, fontStress を定義して v-bind を設定した style 属性の中で利用しています。
- 文字の色を赤から青に変更したい場合は activeColor の値を red から blue に変更します。
```html
<script setup>
  const activeColor = 'red';
  const fontStress = '900';
</script>

<template>
  <h1>Vue 3 入門</h1>
  <p :style="{ color: activeColor, fontWeight: fontStress }">
    v-bindの設定方法の確認
  </p>
</template>
```
上記では fontWeight をキャメルケースで記述していますが通常の style のプロパティを利用した場合はシングルクォーテーションを使います。
```html
<p :style="{ color: activeColor, 'font-weight': fontStress }">
  v-bindの設定方法の確認
</p>
```
変数側で style を設定することもできます。変数側でスタイルを設定することで template タグがすっきりと見通しが良くなります。
```html
<script setup>
  const styleObjcet = {
    color: 'red',
    fontWeight: 900,
  };
</script>

<template>
  <h1>Vue 3 入門</h1>
  <p :style="styleObjcet">v-bindの設定方法の確認</p>
</template>
```
通常の style 属性のように記述することもできます。
```html
<script setup>
  const styleObjcet = 'color:red;font-weight:900';
</script>

<template>
  <h1>Vue 3 入門</h1>
  <p :style="styleObjcet">v-bindの設定方法の確認</p>
</template>
```



