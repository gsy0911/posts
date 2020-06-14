# 概要
Windowsでpythonのプログラミングしたい！と思って、いろいろ調べたら
Anacondaを使うと簡単に、しかもWindowsの環境をあまり汚すことなくPythonが使えるらしい。

ということで、Anacondaを使ってPythonの環境を作ってみた。
すでにネットにはたくさんの情報があるけれども、
自分への備忘録と今後の内容のために、準備のところを書きます。


# 環境
* Windows 10 version 1703

なんでLinux系じゃないのか、と言われたら新しいPC(or SSD or DualBoot)を買ったり準備したりするのが面倒だから。
いろいろ問題はありそうだけれども、これで行くしかない。。
* CPU : core-i5 4670K

4年くらい前のすこし古い型版ですが簡単なプログラムなら余裕やね。
* GPU : GTX960

Minecraftのために買った。それ以外には使っていないのでこれを機に使えたらいいなと。。。


# Anaconda3をインストール
では早速作業を始めます。まぁ、ここら付近は簡単ですね。

[公式サイト](https://www.continuum.io/downloads) からAnacondaのインストーラーをダウンロードする。
pythonのバージョンは新しい3.6のを選んだ。
なぜなら、どうせAnaconda上で仮想環境を構築するときに、Pythonのバージョンは選べるのでどれでもいいかなと思う。

インストールが完了したら、新しいプログラムに「Anaconda Prompt」があることを確認する。
今後はこの「Anaconda Prompt」を使って色々していく。
画像の左側（黒色）が通常のコマンドプロンプトで、右側（灰色）がAnaconda Prompt。

![CondaPrompt](https://www.dropbox.com/s/8ap5t8xmv5fkrbb/conda_prompt.PNG?dl=1)


ただ、Windows標準のコマンドプロンプトでもcondaコマンドを打つことができて、
その場合はWindowのユーザー環境変数に以下の3つを追加する。
インストール先を変更していない場合は、[UserName]直下にAnaconda3というディレクトリがあるはず。
それ以外の場合は、順次変更する。

```path:path
C:\Users\[UserName]\Anaconda3
C:\Users\[UserName]\Anaconda3\Scripts
C:\Users\[UserName]\Anaconda3\Library\bin
```


# Anaconda上に仮想環境を構築
「Anaconda Prompt」を起動して、その環境（？）上でいろいろ操作をしてもいいのだけれど、
そうすると、Anacondaのroot環境が汚れてしまうので、仮想環境を構築する。
コマンドプロンプト上にて

```cmd:cmd
> conda create -n [env name] python=3.5
```

などと入力するとpythonのバージョンが3.5の[env name]という環境が構築される。
このとき[env name]は適当な名前を付ける。

作成が完了したら

```cmd:cmd
> conda info -e
```

で作った仮想環境の一覧を見ることができて、

```cmd:cmd
> activate [env name]
```

で作成した仮想環境に入ることができる。
Anaconda Promptの表示が以下のようになったら、入れている。

```cmd:cmd
([env name]) >
```


# PyCharmと連携
あまり使わないかもしれないけど、備忘録として一応。
プロジェクトを作成し、「File」->「Settings」->「Project:\[name\]」->「Project Interpreter」において
Anacondaの仮想環境のディレクトリ内のpython.exeを参照する。

# Jupyterを使う
まずは、仮想環境にて以下のコマンドを打ちjupyterのインストールを行う。

```cmd:cmd
> pip install jupyter
```

そうしたら、jupyterを起動する

```cmd:cmd
> jupyter notebook
```

と入力すれば、ブラウザからログインできる。

細かい使いかたなどは [ここ](http://pythondatascience.plavox.info/python%E3%81%AE%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%83/jupyter-notebook%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%88%E3%81%86)を参考にしてどうぞ。
