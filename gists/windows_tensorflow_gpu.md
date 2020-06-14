# WindowsでTensorFlow(GPU)をつかう
* 更新日時 : 2017年8月5日


# CUDAとcuDNNをダウンロードとインストール

* CUDA 8.0 : [https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads) 
* cuDNN 6.0 : [https://developer.nvidia.com/cudnn](https://developer.nvidia.com/cudnn)

とくに何事もなくインストールはでき，またパッチ２（2017年8月4日現在）も出ていたのでそれを充てた．
インストール中は，画面が点滅したりする．
パッチは，当然だけれども，すぐにインストールが完了した．
CUDAのPATHは自動で追加されているみたいなので，
CUDAのインストール先にcuDNN.zipを中身である「cuda」を入れた．

自分の場合は，以下のディレクトリの直下に入れた

```path:path
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0
```

公式サイトの

```cmd:cmd
> pip install --ignore-installed --upgrade https://storage.googleapis.com/tensorflow/windows/gpu/tensorflow_gpu-1.2.1-cp35-cp35m-win_amd64.whl
```
では失敗した．ので参考サイトを見てみると，


```cmd:cmd
> pip install tensorflow-gpu
```

で入れることができた．
そして，起動を確認してみるために，やろうとしたら，エラーを吐かれた．．．

"common problem"であるようなので，[https://www.tensorflow.org/install/install_windows](ここ)をみて解決を試みた．
* No module named "pywrap_tensorflow"



# 参考にしたサイト
* [http://qiita.com/binzume/items/7a0170fe834e250ddef9](WindowsでGPU使ってTensorFlowを動かすメモ(TensorBoardも動かす))
* [http://qiita.com/JUN_NETWORKS/items/cc78f0c55c67081b44a7](Windows環境でTensorFlow-GPUをpipインストールするまで)
* [http://yaju3d.hatenablog.jp/entry/2016/12/06/023820](TensorFlowがWindowsサポートしたのでインストールしてみた)
* [http://kz-engineer-scrap.hatenablog.com/entry/2017/05/14/013303](jupyter で ModuleNotFoundError: No module named 'tensorflow' エラー)