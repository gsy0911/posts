# TL;DR

**GitPodを利用すると、iPadでIDEもjupyterも使えるから最高やんけ！**

# GitPodの特徴
`GitPod`は以下のような特徴があります。

* VS codeライクなIDEを利用可能
* chromeとfirefoxで使用可能
* `GitHub`とアカウント連携して利用可能
* publicリポジトリは無料、privateリポジトリは有料(9$/月〜)で利用可能


# 今回のゴールと手順

iPadのchrome上で
Pythonのプログラミング環境（IDEとjupyterlab）構築をします。
大まかな手順は以下の通りです。

1. GitHubにリポジトリを作成し、3つのファイルを置く
2. GitPodへアクセス
3. jupyterlabを立ち上げる

## GitHubでの準備

適当なリポジトリに以下の3つのファイルを置きます。
1ファイル目は、GitPod上で利用するdockerファイルです。


```.gitpod.Dockerfile
FROM python:3.7
RUN apt update -y && apt upgrade -y
RUN pip install pipenv
```

2ファイル目は、`GitPod`の設定ファイルです。
8888番ポートを開いている理由は`jupyterlab`を利用するためです。

```.gitpod.yml

image:
  file: .gitpod.Dockerfile

ports:
- port: 8080
  onOpen: open-preview
- port: 8888
  onOpen: open-browser
```

3ファイル目は`Pipfile`です。
これは、`pipenv`で仮想環境を構築していることを想定して準備したファイルのため必須ではありません。（pipenvを利用するとローカルの環境などをそのままGitPod上でも利用できるのでオススメです。）パッケージは`jupyterlab`と`pandas`などがあればいいかなと思います。


```Pipfile:Pipfile
[[source]]
name = "pypi"
url = "https://pypi.org/simple"
verify_ssl = true

[dev-packages]

[packages]
pandas = "*"
jupyterlab = "*"
matplotlib = "*"

[requires]
python_version = "3.7"
```

ここまでで、以下の3ファイル（とREADME.md）がリポジトリの直下に配置されていると思います。

```github:github_repository
(リポジトリルート)
 |-.gitpod.Dockerfile
 |-.gitpod.yml
 |-Pipfile
 |-README.md
```

## GitPodへアクセス

GitPodへのアクセスはGitHubリポジトリのURLに`gitpod.io/#`を付与するだけです。
たとえば、GitHubリポジトリのURLが以下だった場合、

```
https://github.com/[user_name]/[repository_name]
```


`GitPod`にアクセスするURLは次のようになります。


```
https://gitpod.io/#github.com/[user_name]/[repository_name]
```

毎回打ち込むのは大変なのでGitHubのREADME.mdなどにリンクを貼っておくと便利です。

GitPodのIDEは、初回は起動に10分程度かかりますが、2回目以降は1~2分で起動します。



## jupyterlabを立ち上げる。
リポジトリに、`Pipfile`があるので、以下のコマンドを叩くだけでjupyterlabを立ち上げることができます。

```bash
$ pipenv install
$ pipenv shell
([repository_name])$ jupyter lab
```

pipenvのshellを立ち上げて、

![6B17E2BE-C039-4DC3-96F2-024DD99E64AF.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/d9734748-2e0c-d718-d802-c9cae72fb893.png)

jupyterlabも起動・ログインがiPad上で操作できます。

![C4077621-FBFC-45D7-8E53-46FC77184D6A.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/c3a08c99-b43f-80fb-5cbb-f171c3acf61f.png)

## iPadでGitPodを利用するTips

iPadでGitPodを利用していると、画面下に予測候補などが現れる長いバーが出てきます。（画像下部の2色になっている黒いバー）
![2181F806-DBE6-4112-98C1-EE7BFC7E74E6.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/3743e93c-ace2-fb76-9c54-e2b8e0b6a717.png)

これは、iPadの「設定」→「一般／キーボード」の「入力補助」をオフにすると消えてくれます。
![56400627-9BA2-4699-BE6F-16A515FD6BDF.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/019d37e3-c7df-782e-ecb4-774510ce26e1.png)


## おわりに

iPadで出来ることがどんどん増えてきているので、
簡単なプログラミングなら、本当にiPadだけで十分だなと思う今日この頃です。
（ちなみに、この記事もiPadで書いてます。）

# 参考

* [Gitpodが最強過ぎる件について](https://qiita.com/mouse_484/items/394a4984f749cc201422)
* [Gitを直接編集できるVSCodeライクなCloudIDEがあるらしい(GitPod)](https://qiita.com/yuzukaki/items/d127f8f425b088cc3785)
* [gitpodでjupyternotebookの開発環境を作る](https://qiita.com/ymmmtym/items/14de67dc64b3c47704e8)




