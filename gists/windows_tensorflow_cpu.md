# WindowsでTensorFlow(CPU)をつかう
* 更新日時 : 2017年8月5日

話題になっているディープラーニング

個人には手を出しにくい？

windowsでもできるようになった．

ゲームの普及で高性能GPUが入っているやろｗ

そこで，windows環境でDeepLearning環境の1つであるTensorFlowを使ってみました．

# 環境
* OS : Windows 10 (64bit) - version : 1703
* CPU : core-i5 4670K
* Anaconda3
* Python : 3.5.2


# Anacondaを使ってPythonのインストール
Anacondaをインストール

そしてAnaconda Promptを起動して

```console:anaconda prompt
> conda create -n [env-name] python=3.5
```
と入力してPythonの仮想環境を作成します．
この環境には，anaconda promptから

```console:windows
> activate [env-name]
```

で入ることができ，また

```console:windows
> deactivate [env-name]
```

でログアウトすることができる．

# まずはCPU版をインストール

仮想環境に[tf_cpu]を作る．

```cmd:cmd
> conda create -n tf_cpu python=3.5.2
```

そうしたら，以下のコマンドを打ってtensorflowをインストールし，
バージョンを確認してみた．

```cmd:cmd
> pip install tensorflow
> conda list | grep tensorflow
tensorflow 1.2.1 <pip>
```

バージョンは1.2.1でpipを使ってインストールされたことが確認できる．
次に，tensorflowが正しくインストールされているかを確認する．
公式サイトに書かれている方法で試してみる．

```cmd:cmd
> python
>>> import tensorflow as tf
>>> hello=tf.constant('Hello, TensorFlow!')
>>> sess=tf.Session()
>>> print(sess.run(hello))
b'Hello, TensorFlow!'
```
とりあえず動いた．


# 参考にしたサイト
