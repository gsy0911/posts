# はじめに

ウェブの知識がなさ過ぎて辛みしかない！
ということで，ウェブをちょっと触ってみた！！

## 作成物

* 作るウェブアプリは「ガチャシミュレーター」
* 引く回数を指定して「SSR」と「SR」と「N」が何回出るか表示する

## 完成物

* ガチャを引く前の画面

![capsul_toy_10.PNG](https://qiita-image-store.s3.amazonaws.com/0/260295/2eb606ad-e176-1089-4b4f-11c90905ca66.png)


* ガチャを引いた後の画面

![capsul_toy_pull.PNG](https://qiita-image-store.s3.amazonaws.com/0/260295/3de35e88-37cd-24b4-cd3b-4182b9e3657a.png)

(背景が白だからページが分かりづらい．．)

## 動作環境

環境は以下の通り

| フロント | バックエンド | サーバー | インフラ |
|:--:|:--:|:--:|:--:|
| [Vue.js](https://jp.vuejs.org/index.html) | - | [nginx](https://nginx.org/en/) | [GCP](https://cloud.google.com/) |

とりあえず，サイトにアクセス（IP直打ち）したらページが表示されるところまでを目指した．

# 操作手順

## GCPにログインしてVMインスタンスを立ち上げる

たぶんここら付近はググればたくさん出てくるので割愛した．
画面キャプチャ付きでたくさん情報が出ていると思うので．．．

## nginxのインストールと設定

以下のコマンドを打ち`nginx`をインストールする．

```
$ sudo apt install nginx
```

そうしたら，デフォルトで表示されるページをコピーして`~/app`ディレクトリに持ってきて編集できるようにする．

```
$ mkdir app
$ cp /var/www/html/index.nginx-debian.html ./app
```

次に，nginxの設定ファイルで移動させたhtmlを初期ページに設定する．

```nano:/etc/nginx/sites-enabled/default
~~前略~~

#下の行をコメントアウトして
#root /var/www/html;

#以下の行を追加する
root /home/[your_name]/app;

~~後略~~
```

ここで，GCPに設定されているIPにアクセスして表示されていたらOK.

## ページ作成

移動してきた`index.nginx-debian.html`を編集する

```html:~/app/index.nginx-debian.html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"/>
<title>Simulator</title>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/uikit/3.0.0-rc.25/css/uikit.min.css"/>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
<h1 style="text-align: center">Capsule Toy Simulation</h1>

<div id="app" style="text-align: center">
<div v-if="first"  style="text-align: center">
 pull <select v-model="selected"  class="uk-select uk-form-width-xsmall">
  <option v-for="option in options" v-bind:value="option.value">
  {{ option.text }}
  </option>
 </select> times
 <br><br><br><br>
 <button v-on:click="pull" class="uk-button uk-button-primary">PULL</button><br>
</div>

<div v-if="second" style="text-align: center">
 SSR: {{ ssr }} <br>
 SR : {{ sr }}<br>
 N  : {{ n }}<br><br>
 <button v-on:click="back" class="uk-button uk-button-primary">BACK</button><br>
</div>
</div>


<script type="text/javascript">
var app = new Vue({
 el: '#app',
 data: {
  selected: 1,
  first: true,
  second: false,
  ssr: 0,
  sr: 0,
  n: 0,
  options: [
   {text: '1', value: 1},
   {text: '2', value: 2},
   {text: '3', value: 3},
   {text: '4', value: 4},
   {text: '5', value: 5},
   {text: '6', value: 6},
   {text: '7', value: 7},
   {text: '8', value: 8},
   {text: '9', value: 9},
   {text: '10', value: 10}
  ]
 },
methods:{
  pull: function(){
   this.first = false
   this.second = true
   for(var i=0; i < this.selected; i++){
    this.gacha(Math.random())
   }
  },
  back: function() {
   this.first = true
   this.second = false
   this.ssr = 0
   this.sr = 0
   this.n = 0
  },
  gacha: function(prob) {
   if (prob < 0.03 ){
     this.ssr = this.ssr + 1
     return
   }
   if (prob < 0.1) {
     this.sr = this.sr + 1
     return
   }
   this.n = this.n + 1
   return
  }
 }
})
</script>
</body>
</html>
```

今回は，`Vue.js`を使ってみた．
`v-if`，`v-for`属性が便利でhtmlにjsべた書きだけれど`SPA`を実現できた．


# おわりに

とりあえず，今回はGCPを使って何か作ってみたいということで
バックエンドとかを全く意識せずに作ったけれど，次回以降は何らかのバックエンドを実装したい．
候補としては
* `Elixir + phoenix`
* `python + responder`
のどちらかなと思っている．

2018/12/22　追記
* [バックエンド実装しました](https://qiita.com/Lilly008000/items/0efd08c4f5c13d0223cb)
