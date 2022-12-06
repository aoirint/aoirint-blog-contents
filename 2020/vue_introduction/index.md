---
# moved from https://aoirint.hatenablog.com/entry/2020/05/10/221714
title: Vue 入門
date: '2020-05-10T22:17:14+09:00'
draft: false
channel: 技術ノート
category: Vue
tags:
- Vue
- StaticWeb
---
# Vue 入門

Node.js（npm+Browserifyなど）でパッケージ管理したり、サーバサイドで事前処理（`.vue`ファイル、vue-cli）して配信ファイルを生成したりしてややこしいけれど、基本的にVue（Vue.js）はブラウザ上で動作するBootstrap++みたいなクライアントサイドフレームワークという理解。

## 参考
- [はじめに — Vue.js](https://jp.vuejs.org/v2/guide/index.html)

この「はじめに」をベースにサンプルを並べていく。


## 何もしないHTML

```html

<!DOCTYPE html>
<meta charset="utf-8">

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->

<h2>Hello Vue!</h2>
<p>
  This is not Vue!

```

CDNから開発用のVue.jsを持ってくるだけ。これをベースにいじっていく（SRIは略）。

テストするときは`python3 -m http.server --bind 127.0.0.1`とか、`php -S 127.0.0.1:8000`とか、静的コンテンツを開ければなんでもよし。


## テンプレート的用法

### 適当に使ってみる

```html
<!DOCTYPE html>
<meta charset="utf-8">

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->

<h2>Hello Vue!</h2>
<p id="body">
  {{ body_content }}

<script>
(function() {
  var app = new Vue({
    el: '#body',
    data: {
      body_content: 'This is BODY!',
    },
  });
})();
</script>

```

Django/Jinjaのように`{{ variable }}`の形式で埋め込み位置を指定。Vueオブジェクトを作って初期値を与える。


### コンテナ要素で使ってみる

```html
<!DOCTYPE html>
<meta charset="utf-8">

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->

<div id="app">
  <h2>{{ title }}</h2>
  <p>
    {{ body }}
</div>

<script>
(function() {
  var app = new Vue({
    el: '#app',
    data: {
      title: 'Hello Vue!',
      body: 'This is BODY!',
    },
  });
})();
</script>
```

要素の内部に複数の埋め込みを行う。


### 属性をテンプレート化

```html
<!DOCTYPE html>
<meta charset="utf-8">

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->

<div id="app">
  <h2 v-bind:title="tooltip">{{ title }}</h2>
  <p>
    {{ body }}
</div>

<script>
(function() {
  var app = new Vue({
    el: '#app',
    data: {
      tooltip: 'Hover text',
      title: 'Hello Vue!',
      body: 'This is BODY!',
    },
  });
})();
</script>
```

属性（この場合、ホバーテキスト/ツールチップを表す`title`属性）に埋め込みを行う（`title`という名前が2つ出てきてややこしくなってしまった）。


### スクリプトからの書き換え

```html
<!DOCTYPE html>
<meta charset="utf-8">

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->

<div id="app">
  <h2 v-bind:title="tooltip">{{ title }}</h2>
  <p>
    {{ body }}
</div>

<script>
(function() {
  var app = new Vue({
    el: '#app',
    data: {
      tooltip: 'Hover text',
      title: 'Hello Vue!',
      body: 'This is BODY!',
    },
  });

  app.title = 'This is TITLE!';
  app.tooltip = 'This is TOOLTIP!';

})();
</script>
```

埋め込みテキストはスクリプトから変更可能（すぐに反映される）。


## イベントハンドリング

### クリックイベント

```html
<!DOCTYPE html>
<meta charset="utf-8">

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->

<div id="app">
  <h2 v-bind:title="tooltip">{{ title }}</h2>
  <p>
    {{ body }}
  <p>    
    <button v-on:click="onButtonClicked">Switch</button>
</div>

<script>
(function() {
  var app = new Vue({
    el: '#app',
    data: {
      tooltip: 'Hover text',
      title: 'Hello Vue!',
      body: 'This is BODY!',
    },
    methods: {
      onButtonClicked: function() {
        this.title = 'This is TITLE!'; // this == app
        this.tooltip = 'This is TOOLTIP!';
      },
    },
  });

})();
</script>
```

クリックイベントが起きたとき埋め込みテキストを書き換える。


## Vueの内部状態とinputタグ

```html
<!DOCTYPE html>
<meta charset="utf-8">

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->

<div id="app">
  <p>
    <input v-bind:value="field1">
  <p>
    <input v-model="field2">
  <p>
    <button v-on:click="check">Check</button>
</div>

<script>
(function() {
  var app = new Vue({
    el: '#app',
    data: {
      field1: 'Field 1',
      field2: 'Field 2',
    },
    methods: {
      check: function() {
        console.log(app.field1);
        console.log(app.field2);
      },
    }
  });

  app.check();

})();
</script>
```

Vueの内部状態と画面上の状態がずれたときの挙動を試す。`v-bind:value`では再描画が走ったときに値がVueの内部状態にリセットされる。これを防ぐ（Vueの内部状態と表示状態を同期する）には`v-model`を使う（双方向バインディング）。


## スタイル操作

### Visibility

```html
<!DOCTYPE html>
<meta charset="utf-8">

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->

<div id="app">
  <p>
    <button v-on:click="onButtonClicked">Switch</button>
  <p v-if="message_visible">
    Hello Vue!
</div>

<script>
(function() {
  var app = new Vue({
    el: '#app',
    data: {
      message_visible: true,
    },
    methods: {
      onButtonClicked: function() {
        app.message_visible = !app.message_visible;
      },
    }
  });

})();
</script>
```

`display: none`の切り替え。条件分岐という扱いらしい。

```html
<!-- Django / Jinja -->
{% if message_visible %}
<p>
  Hello Vue!
{% endif %}
```

これの代わりなのはわかるが、タグ名が先に来ているのが読みにくい..（Rubyか？）。


## 再利用

### 配列のマッピング

```html
<!DOCTYPE html>
<meta charset="utf-8">

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->

<div id="app">
  <ol>
    <li v-for="item in items">
      {{ item.text }}
  </ol>
</div>

<script>
(function() {
  var app = new Vue({
    el: '#app',
    data: {
      items: [
        { text: 'Item 1' },
        { text: 'Item 2' },
        { text: 'Item 3' },
      ],
    },
  });

})();
</script>
```

`v-for`を付けた要素をリストに基づいて複製して個数分表示する。これもタグ名が先に来ているのが読みにくい..。

```html
<!-- Django / Jinja -->
<ol>
{% for item in items %}
  <li>
    {{ item.text }}
{% endfor %}
</ol>
```

これの代わりなのはわかる。


### コンポーネント
新しいタグ名を定義する感覚で要素の再利用（コード上での定義）ができる。

```html
<!DOCTYPE html>
<meta charset="utf-8">

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->

<script>
  Vue.component('my-item', {
    template: '<li>Item</li>',
  });
</script>

<div id="app">
  <ol>
    <my-item v-for="item in items"></my-item>
  </ol>
</div>

<script>
(function() {
  var app = new Vue({
    el: '#app',
    data: {
      items: [
        { text: 'Item 1' },
        { text: 'Item 2' },
        { text: 'Item 3' },
      ],
    },
  });

})();
</script>
```

閉じタグ省略したら警告出た。`text`をちゃんと使うには、以下のようにする。

```html
<!DOCTYPE html>
<meta charset="utf-8">

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->

<script>
  Vue.component('my-item', {
    props: [ 'my_item', ],
    template: '<li>{{ my_item.text }}</li>',
  });
</script>

<div id="app">
  <ol>
    <my-item
      v-for="item in items"
      v-bind:my_item="item"
      ></my-item>
  </ol>
</div>

<script>
(function() {
  var app = new Vue({
    el: '#app',
    data: {
      items: [
        { text: 'Item 1' },
        { text: 'Item 2' },
        { text: 'Item 3' },
      ],
    },
  });

})();
</script>
```

`v-bind:text="item.text"`みたいなこともできるらしい。


## Vue CLIって何

今回は単一のHTMLファイルに全部書いてるが、巨大なプロジェクトではそうもいかない。
コンポーネントの定義なんかは外部のJSを持ってくればよさそうだけれど、これも数が多くなると管理がつらくなってくる。
このへんをなんかいい感じ（コンポーネントごとにファイル分けたり）にして、ついでにURL（パス、ルーティング）もいい感じにできる（JSでルーティング設定）ようにして、最終的にブラウザ側でVueの処理をいい感じにできるように整えた静的サーバコンテンツ（GitHub Pagesとかで動くやつ）を生成するツールを作ってみた、というのがVue CLIっぽい。
