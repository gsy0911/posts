# はじめに

本記事は、パッケージの作成から`PyPI`に登録までの方法を記します。
どちらかというと、備忘録的な感じです。

# 自作パッケージ公開までの流れ

自作パッケージを公開するまでの流れは以下の通りです。
６ステップで`PyPI`に公開できます。
（この中で一番大変なのは、(1 )のパッケージ作成の部分です）

1. パッケージ作成（`Python`でコーディング）
2. pip でインストールできるように設定ファイルの追加
3. 配布物の作成
3. `test-PyPI`にテスト登録
4. `PyPI`に登録
5. deploy の自動化

# 1. ライブラリ作成

`<package_name>`にはライブラリの名前にしておくといいです。
[numpy](https://github.com/numpy/numpy)や[pandas](https://github.com/pandas-dev/pandas)、[scikit-learn](https://github.com/scikit-learn/scikit-learn)などはこの形式です。
ただ、[matplotlib](https://github.com/matplotlib/matplotlib)は、`src`になっているなど例外もあります。

```bash
├─ <package_name>/
|  └─ __init__.py
└─ README.md
```

そして、この`<package_name>`直下にある`__init__.py`で宣言された関数や変数が、
`import`後利用できるようになります。`pandas`の例で言うと、以下の通りです。

```python
import pandas as pd
pd.read_csv("...")
```

として、`pd.read_csv()`が利用できるのは、`__init__.py`の中に関数が宣言されているためです。
（厳密には、`__init__.py`の中では`read_csv()`そのものは定義されていません）。


例として、`Hoge`クラスを定義しておきます。

```python:__init__.py
class Hoge:
    def __init__(self):
        pass
```

このようにしておくと、以下のように利用できます。

```python:main.py
import <package_name>

hoge = <package_name>.Hoge()
```


# 2. 設定ファイルの追加

適当なライブラリのクラスを定義・作成したら、次は公開の準備をすすめていいきましょう。
最低限必要なファイルは`setup.py`のみです。

`setup.py`は、パッケージの情報を記載します。
詳しくは「[PyPIパッケージ定義ファイル作成方法 - __init__.py setup.py MANIFEST.in の書き方](https://qiita.com/shinichi-takii/items/6d1063e0aa3f79e599f0)」をご覧ください。


```python:setup.py
import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()


setuptools.setup(
    name="<package name>",
    version="<version>",
    author="<your name>",
    author_email="<your email>",
    description="<short_description>",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/<your_github_name>/<repository_name>",
    packages=setuptools.find_packages(),
    install_requires=[
        "pandas>=1.0.0",
        "s3fs>=0.4.2"
    ],
    license="MIT",
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires='>=3.7',
)
```

また、必要に応じて`MANIFEST.in`を作成します。
これは、配布するパッケージに含める／除くファイルを決められます。

```txt:MANIFEST.in
include MANIFEST.in
include setup.py
include README.md
```

`setup.py`と`MANIFEST.in`を追加したあとのファイル構成は以下の通りです。

```bash
├─ <package_name>/
|  └─ __init__.py
├─ setup.py
├─ MANIFEST.in (必要に応じて)
└─ README.md
```


# 3. 配布物の作成

`setup.py`を作成したら、いよいよ`PyPI`に登録していきます。
配布用のパッケージ作成と`PyPI`にアップロードするように以下のパッケージを install します。

```shell
$ pip install twine wheel
```

次のコマンドを実行し、ソースコード配布物を作成し、ライブラリのパッケージを作成します。

```shell
$ python setup.py sdist bdist_wheel
```

すると、以下のようにフォルダ作成されます。

```bash
├─ <package_name>/
|  └─ __init__.py
├─ <package_name>.egg-info/
├─ dist/
├─ setup.py
├─ MANIFEST.in (必要に応じて)
└─ README.md
```

それぞれのファイルは、以下の通りです。

>* <パッケージ名>.egg-info ディレクトリ
ソースコード配布物作成時の中間ファイルが書き出されるディレクトリです。
MANIFEST.in から配布物に含めるファイルを削ったとき、それだけでは反映されず、同ディレクトリのファイルを一度削除してリセットする必要があります。

> * dist ディレクトリ
パッケージファイルが書き出されるディレクトリです。
パッケージファイルは、同ディレクトリに追加作成されます。
PyPIに不要なファイルをアップロードしてしまわないよう、最初にファイル削除することをお勧めします。




また、パッケージをアップデートする際には以下のコマンドを打ち、
以前のバージョンの配布物が入ってる`<package_name>.egg-info`と dist 以下の中身を消すようにしましょう。
これらのファイルは再生成できるため削除してしまって構わないです。

```shell
$ rm -f -r <package_name>.egg-info/* dist/*
```



# 4. `test-PyPI`に登録・アップロード

配布物の作成が終わったら、[test-PyPI](https://testpypi.python.org/)にテスト登録をしてみましょう。
`test-PyPI`でアカウントを作成し、以下のファイルをホームディレクトリに作成します。
（以下ファイルには本番`PyPI`の情報も記述してあります）

```txt:~/.pypirc
[distutils]
index-servers =
  pypi
  testpypi

[pypi]
repository: https://upload.pypi.org/legacy/
username: <本番用アカウント名>
password: <パスワード>

[testpypi]
repository: https://test.pypi.org/legacy/
username: <テスト用アカウント名>
password: <パスワード>
```

`~/.pypirc`の作成が終わったら、次のコマンドを実行し、`test-PyPI`に登録します。

```
$ twine upload -r testpypi dist/*
```

`test-PyPI`に登録ができたら、`pip`でインストールできるか確認しましょう。

```shell
$ pip install -i https://test.pypi.org/simple/ <package_name>
```


また、`poetry`を利用している場合は以下の`pyproject.toml`に以下を追加します。

```text:pyproject.toml
+ [[tool.poetry.source]]
+ name = "testpypi"
+ url = "https://test.pypi.org/simple/"
+ secondary = true

[tool.poetry]
name = "python_env"
version = "0.1.0"
description = ""
authors = ["your_email"]

[tool.poetry.dependencies]
python = "^3.7"
pandas = "^1.0.3"

[tool.poetry.dev-dependencies]

[build-system]
requires = ["poetry>=0.12"]
build-backend = "poetry.masonry.api"
```

そうすると、通常の`poerty add`コマンドでインストールできるようになります。


```
$ poerty add <package_name>
```

この`test-PyPI`を利用して、`pip`でインストールできるか確認すると良いです。
ちなみに、`test-PyPI`に登録して実際に`pip`でインストールできるようになるまで最大で１０分程度かかりました。


# 5. PyPIに登録

`test-PyPI`から正しく`pip install`ができたら、[https://pypi.python.org/](https://pypi.python.org/)に登録しましょう。
まずは、`test-PyPI`と同様にユーザーアカウントを作成し、`~/.pypirc`にアカウント情報を追記します。

すでに、配布物は作成済みなので、以下のコマンドを実行するだけです。

```
$ twine upload -r pypi dist/*
```

実行後は、`PyPI`に登録されているか確認後、実際に`pip install`してみましょう。
（poerty などで実行する場合は、追加した url を削除しておきましょう）

`test-PyPI`に登録が成功すれば、`PyPI`への登録もすぐできます。

# 6. deployの自動化

毎回、同じコマンドを実行するのは大変なので
１つのコマンドで自動削除・配布分作成・`PyPI`登録してくれるように`Makefile`を作成しましょう。

※この時、`:`の後に続くのは`space`ですが、先頭から文字を入力する場合は、`tab`じゃないと`Makefile:-: *** missing separator.  Stop.`というエラーが出ます。
（Qiita では表示の都合上 space になっているのでそのままコピーしてもエラーになります）

```txt:Makefile
.PHONY: deploy
deploy: build
	twine upload dist/*

.PHONY: test-deploy
test-deploy: build
	twine upload -r testpypi dist/*

.PHONY: build
build: clean
	python setup.py sdist bdist_wheel

.PHONY: clean
clean:
	rm -f -r azfs.egg-info/* dist/* -y

```

上記のような`Makefile`を作成し、以下のコマンドを実行すると`PyPI`に登録してくれます。

```bash
# テストPyPIの場合
$ make test-deploy

# 本番PyPIの場合
$ make deploy
```

# おわりに

一通り、パッケージ作成から`PyPI`登録までの流れを記事にしてみました。
これで皆さんもパッケージ配布できますね。

## 2020/05/09追記
`poetry`ならもっと簡単にできました。

### `test-PyPI`に登録

```
# test-pypiのurlを登録
$ poetry config repositories.testpypi https://test.pypi.org/legacy/
$ poetry config --list

> cache-dir = "/Users/<user_name>/Library/Caches/pypoetry"
> repositories.testpypi.url = "https://test.pypi.org/legacy/"
> virtualenvs.create = true
> virtualenvs.in-project = true
> virtualenvs.path = "{cache-dir}/virtualenvs"  # /Users/<user_name>/Library/Caches/pypoetry/virtualenvs

$ poetry build
$ poetry publish -r testpypi -u <user_name> -p <password>
```

### `PyPI`に登録

```
$ poetry build
$ poetry publish -u <user_name> -p <password>
```


# 参考

* [PyPIパッケージ公開手順](https://qiita.com/shinichi-takii/items/e90dcf7550ef13b047b5)
* [Twineを使ったPyPI/TestPyPIへのパッケージ更新](https://raimon49.github.io/2018/03/29/register-package-to-pypi-and-testpypi-by-twine.html)
* [poetry adding source](https://python-poetry.org/docs/repositories/)
* [PyPIデビューしたい人の為のPyPI登録の手順](https://qiita.com/kinpira/items/0a4e7c78fc5dd28bd695)
* [Poetryを使ったPythonパッケージ開発からPyPI公開まで](https://kk6.hateblo.jp/entry/2018/12/20/124151)
