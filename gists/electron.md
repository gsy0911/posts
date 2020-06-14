
# 環境

* 接続元 : Windows 10

* raspberry pi 3 B+

```
~/$ uname -a
Linux phoenix 4.14.79-v7+ #1159 SMP Sun Nov 4 17:50:20 GMT 2018 armv7l GNU/Linux

//rpi-upgrade
~/$ uname -a
Linux phoenix 4.19.27-v7+ #1206 SMP Wed Mar 6 14:40:18 GMT 2019 armv7l GNU/Linux
```
* node : v11.2.0

# 手順

## 初期設定など

最初に`electron`の実行に必要なパッケージをインストールしておく。
なお，これらを入れてもエラーが出る場合は，`sudo apt search {package-name}`で調べるとよい。

```
$ sudo apt install -y libnss3-dev　libgtk-3-0 libx11-xcb-dev

```

# 画面を転送

[ここ](https://qiita.com/SSAS3/items/31ea8def8c25ca86af9a)を参考に以下を実行する。

> 1. ターミナルで、sudo raspi-configと入力し、設定画面を開く。
> 2. 5番目のInterfacing Optionsを選択する。すると、P1～P8の8つの項目が表示される。
> 3. P3のVNCがenableになっているかを確認し、なってなかったらenableにする。

そしたら，`xrdp`をインストールする。

```
$ sudo apt install xrdp
```

ターミナルから`ssh -X [user_name]@[hostname]`でつなぎます。

### installation

ここからアプリのフォルダに移動する。

```
~/electron/test$ npm install i -D electron@3.0.13 --platform=linux --arch=armv7l
..$ ls -l
total 48
drwxr-xr-x 136 pi pi  4096 Mar 10 13:05 node_modules
-rw-r--r--   1 pi pi 42024 Mar 10 13:05 package-lock.json
$ npm init
$ mkdir src
$ cd src
```

## 3つのファイルを作成する

## 実行

```
~/electron/test$ npx electron ./src
```

# 参考

* [最新版で学ぶElectron入門 – HTML5 でPCアプリを開発する利点と手順](https://ics.media/entry/7298)
* [Electronで作ったアプリケーションをRaspberryPiで動かす方法](https://qiita.com/sublimer/items/815bdb0f8e68fbe63ea3)
* [無いパッケージ](http://igatea.hatenablog.com/entry/2018/02/11/004142)
* [electron error with latest version](https://github.com/electron/electron/issues/16205)