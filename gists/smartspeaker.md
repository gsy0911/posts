# 目的

リモコンをこの世から排除しつつ，コンピュータに操られたい．

# 概要

* 声でお家の家電を操作する
* 通知を声で受け取る
* 声でタスクの指示をされよう

イメージ図的な？

# 準備するもの

* Google Home
* Nature Remo
* 常時起動しているPC

# 準備

今回は，Windows 10にnode.jsを入れてやる．
そのままwindowsにインストーラーを使ってやろうとしたら，
python環境が必要と言われたのでanacondaを入れて仮想環境上でやることにした．

## node.jsの環境構築

まずは，anacondaを入れる．
とりあえず，python3.6上でやりたかったが，
google-home-notifierの対応pythonのバージョンが
2.5以上3.0より下という制約だったのでpython2.7の環境を構築．

```cmd::cmd
> conda create -n ghn python=2.7 anaconda
```

とコマンドを打って適当にanaconda上にて仮想環境を構築する．
また，Node.jsはバージョンが新しいものではないと動かなかったので
[公式サイト](https://nodejs.org/ja/)からインストーラーをダウンロードして
windows10上に入れた．

## 諸環境の設定

以下のプログラムは同一ネットワーク内にあるgoogle homeを探すのに必要なソフト類．
1. [Bonjour SDK](https://developer.apple.com/download/more/?=Bonjour SDK for Windows)を入れる
2. [Bonjour Print Service](https://support.apple.com/kb/DL999?locale=ja_JP&viewlocale=ja_JP)をいれる


## google-home-notifier
終わったら，anaconda環境上にgoole home notifierのインストールを開始

```cmd::cmd
(ghn) > npm install google-home-notifier
```

無事にインストールが終わったら

```js::main.js
const googlehome = require('google-home-notifier')
const language = 'ja';

googlehome.device('Google-Home', language); 
googlehome.notify('こんにちわ，世界', function(res) {
  console.log(res);
});
```

というプログラムを書いて

```cmd::cmd
(ghn) > node main.js
```

で起動すると，すこし鈍い音がした後にしゃべってくれる．


## IFTTTの連携

# 参考文献
* [Windowsでnpm installしてnode-gypでつまずいた時対処方法](https://qiita.com/AkihiroTakamura/items/25ba516f8ec624e66ee7)
* [[nodejs]google-home-notifierのインストール](https://hiro99ma.blogspot.jp/2017/10/nodejsgoogle-home-notifier.html)
* [Windowsでgoogle-home-notifierを使う](https://qiita.com/tomatosum/items/046200efe9acd9fb00be)
* [Node.js / npmをインストールする（for Windows）](https://qiita.com/taiponrock/items/9001ae194571feb63a5e)
* [Twitterの通知をGoogle Homeに教えてもらう](https://kotodama.today/?p=744)
* [GoogleHomeスピーカーに外部からプッシュして自発的に話してもらいます](https://qiita.com/azipinsyan/items/db4606aaa51426ac8dac)
* [Google Home開発入門 / google-home-notifier解説](https://qiita.com/SatoTakumi/items/c9de7ff27e5b70508066)
* [スマートスピーカー Advent Calendar 2017](https://qiita.com/advent-calendar/2017/smart-speaker)
* [Androidの通知を全部Slackに流してPCでも検知する](http://kirimin.hatenablog.com/entry/2016/05/08/220253)
* [ngrokを使ってお手軽に開発環境のWebサーバを外から接続できるようにしよう](http://tech.misoca.jp/entry/2015/09/04/151451)
* [Google Homeで時報を知らせる](https://qiita.com/udon11/items/fef44cec7b243f93151b)