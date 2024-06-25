---
title: Vue.js 完全ガイド2024を受講して学んだこと
tags:
  - Vue.js
private: false
updated_at: '2024-06-25T15:02:51+09:00'
id: b3711bb26ba960502c23
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

本記事は、Udemy の「[超 Vue.js 完全ガイド 2024 (Vue Router, Pinia 含む)](https://www.udemy.com/course/vue-js-complete-guide)」を受講して学んだこと（基本的な記述方法など）をまとめていこうと思います。
主に、Composition API を使った内容をまとめており、Options API の内容はまとめておりません。

# 仕組みと構成について

Vue.js は、「コンポーネント」と呼ばれる vue ファイル（単一ファイルコンポーネント）作成し、それをうまく配置していく事によって、UI を構築していく。

[Vite](https://ja.vitejs.dev/)を使用することで、ブラウザが解析できるように vue ファイルを javascript に変換してくれる。 ※Vite には他にも色々機能があるようだが、ここでは、Vue.js を使って UI を開発できるようにするためのものという理解に留める。

Vite で開発されるアプリケーションは、`index.html` がベースとなっており、`index.html`から`/src/main.js`が呼び出されて実行されている。

```javascript:main.js
import './assets/main.css'

import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

`createApp()`の引数に指定されたコンポーネントをもとに、UI を作成している。
`createApp()`はあくまで「作成している」だけで、「表示する」ことはしていない。
`mount()`が「表示する」ことを実現しており、どこに表示するのかを引数で指定している。
上記のコードの内容は、「`id='app'`となっている要素の中に`App`コンポーネントを表示する」という流れになっている。

## vue ファイルの構成について

下記は、コンポーネントの javascript を実装する部分である。
必須ではない。

```vue
<script setup>
// javascriptを実装する部分
</script>
```

下記は、コンポーネントの HTML を実装する部分である。
template タグは必須。

```vue
<template>
<!-- htmlを実装する部分 -->
</template>
```

下記は、コンポーネントの CSS を実装する部分である。
必須ではない。

```vue
<style>
/* cssを実装する部分 */
</style>
```

## script　内の値を template 内で使用する

template タグ内のマスタッシュ構文で、script 内の値を使用することができる。

```vue: 値の表示例
<script setup>
const sample = 'hoge'
</script>

<template>
  <h1>{{ sample }}</h1>　<!-- hogeが表示される -->
</template>
```

## script　内の関数を template 内で使用する

クリックされたときに、script 内で定義したメソッドを実行したい場合、`@click`に関数名を指定することで、実現することができる。

```vue: 関数の使用例
<script setup>
let price = 9

function increment() {
  price += 1
  console.log(price) // consoleに値が表示される。ここではpriceに+1された値が表示される。
}
</script>

<template>
  <h1>{{ price }}</h1>　<!-- ※注意：increment()で変数をインクリメントしているが、ここの表示は変わらない -->
  <button @click="increment"> + </button>
</template>
```

# リアクティブな値について

リアクティビティとは、とある変数が更新されたときに、その変数を使用している部分も更新されるような動きのこと。
上記の例では、`increment()`で`price`という値を更新したら、h1 タグで表示(使用)している`price`も更新されて欲しい、のような直感的な動作を「リアクティビティ」と呼んでいる。

## ref 関数を使ってリアクティブにする

上記の例を、リアクティビティに実装したい場合は、`ref`関数を使用して、値をオブジェクトとして管理する。

```vue:リアクティビティな実装例
<script setup>
import { ref } form 'vue'

let price = ref(9)

function increment() {
  price.value += 1 // script内でリアクティブな値を更新する場合'.value'をつける必要がある
}
</script>

<template>
  <h1>{{ price }}</h1> <!-- ボタンを押すたびにインクリメントされた値が表示される。 -->
  <!--　※1：template内では、'.value'は不要。 -->
  <button @click="increment"> + </button>
</template>
```

`ref`を使って値を定義することで、その値が更新されたときに、自動で再レンダリングされるようになるため、リアクティビティが実現されている。
※1：template タグ内では、変数や定数に対して、ref オブジェクトであるかどうかを内部的に判断しているため、`.value`をつけなくて良い。逆につけてしまうと期待する値が表示できない。

javascript でリアクティビティな動きを提供するためには、技術的な問題でリアクティブな値を'オブジェクト'で管理する必要がある。
そのため、上記の例の`price`は、console などで確認すると「ref オブジェクト」となっている。

## reactive 関数を使ってリアクティブにする

下記のように、ref 関数の引数にオブジェクトを指定すると、各プロパティにアクセスする際に`.value`を指定しないといけなくて面倒くさい。。

```vue: オブジェクトをリアクティブにする例1
<script setup>
import { ref } form 'vue'

// リアクティブにしたい値にオブジェクトを指定する
const human = ref({
  name: 'Taro',
  age: 20,
})

console.log(human.value.name) // Taroが出力される

</script>
```

その際に、reactive 関数を使用することで改善できる。

```vue: オブジェクトをリアクティブにする例2
<script setup>
import { reactive } form 'vue'

const human = reactive({
  name: 'Taro',
  age: 20,
})

console.log(human.name) // Taroが出力される

</script>
```

上記のように reactive 関数を使用することで、human は「reactive オブジェクト」となり、各プロパティにアクセスする際は、通常のオブジェクトを扱うかのように、`.value`をつけずに取得できる。
また、reactive オブジェクトのプロパティにアクセスする際には、その値が ref オブジェクトかどうかを内部的に毎回チェックしているため、reactive オブジェクト内に ref オブジェクトの値を定義したとしても、そのプロパティにアクセスする際でも`.value`は不要。

ただし、、、

配列の場合は例外で、配列のとある要素にアクセスする際には、`.value`が必要である。
これは、ref オブジェクトの配列にしても同様である。

```vue: リアクティブにした配列の要素へアクセスする例
<script setup>
import { ref, reactive } form 'vue'

const items1 = reactive([1,2,3])
console.log(items1[0].value)

const items2 = reactive([ref(1), ref(2), ref(3)])
console.log(items2[0].value)

</script>
```

これは、配列内にプリミティブな値とリアクティブな値が混合していた際に、配列に対するメソッド（例：items.reverse()）の挙動がわかりにくくなってしまうことから、配列では`.value`を明示的に指定する必要がある。

## ref() と reactive() はどっちを使用すれば良いのか？

オブジェクトをリアクティブにしたい場合、ref()でも reactive()でも定義できるのだが、reactive()にはいくつかの制限があるようで、[公式](https://ja.vuejs.org/guide/essentials/reactivity-fundamentals.html#limitations-of-reactive)では ref()を使用することが推奨されている。

## 通常のオブジェクトのプロパティを ref() で定義した場合

通常のオブジェクトのプロパティに ref オブジェクトを定義し、そのプロパティにアクセスする際は、script 内でも template 内でも明示的に`.value`を指定する必要がある。

```vue
<script setup>
import { ref } from 'vue'

const human = {
  name: ref('Taro'),
  age: ref(20),
}

console.log(human.name.value) // Taroが出力

</script>

<template>
  <h1> {{ human.name.value }} </h1> <!-- Taroが出力 -->
  <h1> {{ human.age.value }} </h1> <!-- 20が出力 -->
</template>
```

> ※1：template タグ内では、変数や定数に対して、ref オブジェクトであるかどうかを内部的に判断しているため、`.value`をつけなくて良い。逆につけてしまうと期待する値が表示できない。

上述した内容は、各要素に対して先頭の値のみ ref オブジェクトであるかどうかを判断しているため、通常のオブジェクト内にあるプロパティに対しては、ref オブジェクトであるかどうかを判断してくれないため、明示的に指定する必要がある。

# v-で始まるディレクティブ

Vue.js では、`v-`から始まるディレクティブがあり、様々な機能を提供してくれる。

## v-html

文字列で記述した値を HTML として挿入してくれる。

```vue:例
<script setup>
import { ref } from 'vue'

const message = ref('<h1>Hello</h1>')

</script>

<template>
  <div v-html="message"></div> // h1タグが適用されたHelloが出力される
</template>
```

`v-html`に指定する値は、必ず信頼できるデータを使用する必要がある。
XSS 攻撃を受ける可能性があるため、ユーザーから指定される値は使用しないように実装する必要がある。

## v-bind

タグの属性に動的に値を設定できる。

```vue:例
<script setup>
import { ref } from 'vue'

const vueURL = ref('https://ja.vuejs.org')
const sampleId = ref('sample1')
const id = ref('sample2')

</script>

<template>
  <div v-bind:href="vueURL"></div> // 属性に動的な値を設定する
  <div :id="sampleId"></div> // 「v-bind:」 は 「:」に省略することができる
  <div :id ></div> // 変数名と属性名が一致していれば、さらに省略することができる
</template>
```

## v-on

なにかイベントが発生したときに実行する内容を指定できる。

`v-on:click="xxx"`のように使用。
`@click="xxx"`のように省略が可能。

### インラインハンドラ

処理を直接指定する方法

```vue:クリックしたときに処理を実行する例1
<script setup>
import { ref } from 'vue'

const count = ref(0)

</script>

<template>
  <p>{{ count }}</p>
  <button @click="count++">Button</button>
</template>
```

処理内に`$event`と記述することで、イベントの情報にアクセスできる。

### メソッドハンドラ

関数を指定する方法

```vue:クリックしたときに処理を実行する例2
<script setup>
import { ref } from 'vue'

const count = ref(0)

function countUp() {
  count.value++
}

</script>

<template>
  <p>{{ count }}</p>
  <button @click="countUp">Button</button>
</template>
```

メソッドの第一引数でイベントの情報を取得することができる。

```vue
<script setup>
function countUp(event) {}
```

イベント情報とは別に引数を使用したい場合は、イベント情報は明示的に指定する必要がある。

```vue:イベント情報と引数の例
<script setup>
import { ref } from 'vue'

const count = ref(0)

function countUp(event, i) {
  count.value += i
}

</script>

<template>
  <p>{{ count }}</p>
  <button @click="countUp($event, 1)">Button</button>
</template>
```

### イベント修飾子

#### preventDefault()

デフォルトの挙動を防ぐ。
a タグをクリックすると href のリンクにページ遷移されるが、下記のように preventDefault() をつけることで、デフォルトの挙動（ページ遷移）がされなくなる。

```vue
<template>
    <a href="https://vuejs.org" @click="$event.preventDefault()">Vue.js</a>
</template>
```

下記のように省略可能。

```vue
<template>
    <a href="https://vuejs.org" @click="$event.prevent">Vue.js</a>
</template>
```

#### stopPropagation()

親要素のイベントまで電波させない。
下記の場合、button タグの click イベントで止まり、div タグの click イベントは実行されない。

```vue
<template>
    <div @click="count++">
        <button @click="$event.stopPropagation()">button</button>
    </div>
</template>
```

下記のように省略可能。

```vue
<template>
    <div @click="count++">
        <button @click="$event.stop">button</button>
    </div>
</template>
```

### キー修飾子

`@keyup=""`
キー入力を入力したら実行される。

`@keyup.space.delete=""`
スペース、デリートキーを入力したときに実行される。

## v-model

リアクティブな値を同期する。

```vue
<script setup>
import { ref } from 'vue'

const userInput = ref('')

</script>

<template>
  <p>{{ userInput }}</p> // inputタグで入力した値と常に同じ値が出力される
  <input v-model="userInput" type="text" />
</template>
```

## v-if / v-if-else/ v-else

真偽値によって要素の表示/非表示を実現する。

```vue
<script setup>
import { ref } from 'vue'

const num = ref(0)

</script>

<template>
  <p v-if="num === 0"> 'num'が 0 の場合に表示 </p>
  <p v-if-else="num < 3"> 'num'が2以下の場合に表示 </p>
  <p v-else> 'num'上記の条件以外の場合に表示 </p>
</template>
```

## v-show

真偽値によって要素の表示/非表示を実現する。

```vue
<script setup>
import { ref } from 'vue'

const ok = ref(true)

</script>

<template>
  <p v-show="ok"> 'ok'が true の場合に表示 </p>
</template>
```

v-if と似ているが、v-if は、要素検証で見たときに、要素自体が削除されるが、
v-show は、要素に`display: none;`の style を当てて非表示にする、という消し方の違いがある。

表示/非表示の切り替えの頻度が高い場合は、v-show のほうが、style を変更するだけなので処理が早くて効率的である。
初回のレンダリング時には、v-show を指定している要素も解析されているため、必要なときだけ表示するような表示頻度が低い要素は、v-if を使用した方が、初回のレンダリング時の処理が効率的である。

## v-for

配列の要素やオブジェクトの値を for 文で回して使用する。
同じ要素を繰り返しレンダリングする際は、Vue.js が内部的にそれぞれの要素をユニークなものとして扱うために key 属性を指定する必要がある。

```vue:配列をforで回す例
<script setup>
import { ref } from 'vue'

const fruits = ref(['Apple', 'Banana', 'Grape'])

</script>

<template>
  <li v-for="fruit in fruits" :key="fruit">{{ fruit }}</li>
</template>
```

```vue:オブジェクトをforで回す例
<script setup>
import { ref } from 'vue'

const user = ref({
  name: 'Taro',
  age: 20,
  email: 'taro@smaple.com'
})

</script>

<template>
  <li v-for="(value, key) in user" :key="value">{{ value }}</li>
</template>
```

配列やオブジェクトだけではなく、特定の数値まで for を回すこともできる。

```vue:数字を直接指定してforで回す例
<template>
  <li v-for="i in 10" :key="i">{{ i }}</li>
</template>
```

ちなみに、`in`は、`of`でも良い。

# とある処理を computed() を使ってまとめる

computed 関数内で使用しているリアクティブの値は、常に監視されており（＝リアクティブエフェクト）、リアクティブな値に対して外部からアクセスがあると、computed 内の処理が再実行される。
つまり、少し長めの処理の結果を、リアクティブな状態を保ったまま定数化している。

```vue
<script setup>
import { ref } from 'vue'

const score = ref(0)
const evaluation = computed(() => {
  return score.value > 3 ? 'Good' : 'Bad'
})

</script>

<template>
  <p>{{ evaluation }}</p> // score.valueの値が3以下の場合は'Bad'、3より大きくなったら'Good'が表示される
  <p>{{ score }}</p>
  <button @click="score++"> + </button>
</template>
```

上記は、button をクリックすることで、computed 関数が監視している score.value の値が変更されるため、computed 関数の第 1 引数のメソッドが再実行され、evaluation の値が更新されている。

computed 関数を使用する注意点として、

- 関数外で定義しているリアクティブな値を更新すること
- 非同期の処理

は、副作用を起こすため実装しないようにする必要がある。
例えば、上記の例で、computed 外で定義されている score の値を更新すると、computed 内で使われているリアクティブな値の更新を検知し、computed のメソッドが再実行される、という処理が無限ループしてしまう。

# 「何かが変更されたら、何かの処理をする」といった動作をする

## watchEffect

watchEffect 内でアクセスしているリアクティブな値を監視し、値に変更があれば、関数を再実行する。
computed とほぼ同じ動きをするが、watchEffect の場合は、副作用（非同期処理など）を関数内に入れることができるが、監視対象の値へのアクセスは、非同期処理内などでアクセスしても watchEffect は機能しない。

## watch

watch 関数内でアクセスしているリアクティブな値の内、監視したい値を明示的に指定し、その値に変更があれば関数を再実行する。

第 1 引数には、関数を定義することもできる。その場合は、watchEffect と同じ動きをする。

第 2 引数に定義する関数では、更新後と更新前の値を引数にして関数を実行することができるが、更新後と更新前の値に差分がない場合、関数は実行されない。

# コンポーネントについて

下記のように、components フォルダに再利用可能な状態のコンポーネントを用意し、必要なファイルで import して読み込み、利用することが可能。

src
 ┣ components
 ┃ ┗ CountUp.vue
 ┗ App.vue

```vue: CountUp.vue
<template>
  <h2>Count Up</h2>
</template>
```

```vue: App.vue
<script setup>
import CountUp from './Component/CountUp.vue'
</script>

<template>
  <h1>App</h1>
  <CountUp />
  <CountUp />
</template>
```

上記は、h1 タグの下に CountUp コンポーネントの内容（h2 タグ）を持ってきている。

## props：親コンポーネントから子コンポーネントへデータを受け渡す

親コンポーネント（他のコンポーネントを呼び出している側）から、子コンポーネント（他のコンポーネントから呼び出されてる側）を使用する際に、親から子へデータを受け渡す手段として、props がある。

```vue: ParentComponent.vue
<script setup>
import ChildComponent from './Component/ChildComponent.vue'
import { ref } from 'vue'

const hoge = ref('sample')
</script>
<template>
  <ChildComponent :foo="hoge" />
</template>
```

```vue: ChildComponent.vue
<script setup>
defineProps({
  foo: {
    type: String // propsで受け取る値の型を指定
    default: 'default' // propsのデフォルト値を指定
  }
})

</script>
<template>
  <p>props: {{ foo }}</p> <!-- sample が表示される -->
</template>
```

ただし、props は、読み取り専用であるため、props のデータを直接更新することはできない。
更新する場合は、props で渡しているデータを更新する必要がある。上記の例の場合、　`hoge`の値を更新することで、子コンポーネントにも変更が反映される。

## emit：子コンポーネントから親コンポーネントへデータを受け渡す

子コンポーネントから、親コンポーネントへデータを受け渡す手段として、emit がある。

```vue: ParentComponent.vue
<script setup>
import ResetButton from './Component/ResetButton.vue'
import { ref } from 'vue'

const count = ref(0)
</script>
<template>
  <p>{{count}}</p>
  <button @click="count++"> +1 </button>
  <ResetButton @reset="count = $event" />
</template>
```

```vue: ResetButton.vue
<template>
  <button @click="$emit('reset', 0)">Reset</button> <!-- sample が表示される -->
</template>
```

`$emit('reset')`とすることで、`reset`という名前のイベントを定義することができ、親コンポーネント側で v-on ディレクティブを使うことでそのイベントを使用することができる。
第 2 引数でデータを受け渡すこともできる。

イベントが発生したときの処理は、親コンポーネント側で実装する。

Vue.js は親コンポーネントから子コンポーネントへのデータの流れが基本であり、子コンポーネントから親コンポーネントへのデータの受け渡しは、props ほど簡単にできないようになっている。
双方にデータの受け渡しが簡単にできると、データの流れがわかりにくく保守性を担保することが難しくなっているため、子コンポーネントから親コンポーネントへのデータの受け渡しを、最低限にわかりやすく実現できる機能として emit がある。

# ライフサイクルフックについて

コンポーネントが DOM にマウントされたときやその前のタイミングを検知して、処理を実行するためのメソッド。
処理は非同期的に処理される。

```vue
<script setup>
import { ref, onBeforeMount, onMounted, onBeforeUpdate, onUpdated, onBeforeUnmount, onUnmounted } from 'vue'

const count = ref(0)

// コンポーネントがMountされる前に1番最初に実行される
console.log('script setup') 

// コンポーネントがMountされる前に実行される
onBeforeMount(() => {
  console.log('onBeforeMount')
})

// コンポーネントがMountされた後に実行される
onMounted(() => {
  console.log('onMounted')
})

// コンポーネントが再レンダリングされる前に実行される
onBeforeUpdate(() => {
  console.log('onBeforeUpdate')
})

// コンポーネントが再レンダリングされた後に実行される
onUpdated(() => {
  console.log('onUpdated')
})

// コンポーネントがUnmountされた前に実行される
onBeforeUnmount(() => {
  console.log('onBeforeUnmount')
})

// コンポーネントがUnmountされた後に実行される
onUnmounted(() => {
  console.log('onUnmounted')
})
</script>
<template>
  <p>count: {{ count }}</p>
  <button @click="count++">+1</button>
</template>
```

# 学習を通しての感想

今まで、React を使って開発したことがあり、Composition API の書き方の雰囲気がとても似ていたので、なんとなくやりたいことはできそうだな、といった印象を受けました。
とはいえ、ガッツリと作り込んだ UI を 1 から構築したことはないので、リッチな UI の状態管理やライフサイクルフックを駆使して実装するのは、パッとはできないかな〜と思いました。

とりあえずは、リアクティブなデータの扱い、ディレクティブの使用感に慣れていければ、なんとなく実装ができそうな気がするので、実践的にコードを書いてみたいなと感じました。

React は、単一方向のみのデータの受け渡し（親から子へ）を許可しており、Vue.js では双方向に受け渡しする手段がある、という違いが見られましたが、その他にもいろいろな違いはあるようでした。
それぞれの特徴を掴んで、大規模向けは React、中小規模向けであれば Vue.js などの考察がされており、技術選定の際の知見材料として興味が湧いてきました。

ただ、どちらも「UI を構築する」という目的は変わらないので、できることが似ているのは当然のことなのかな、という気付きもありました。
適正な言語や FW の選定、好みの言語などの基準が私には無いので、もっと多くの言語に触れて知見を溜めたいなと感じました。

# 参考

https://www.udemy.com/course/vue-js-complete-guide

https://vuejs.org
