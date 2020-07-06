# はじめに

本記事では、`sphinx`と`Read the Docs`を利用したドキュメント公開に関する方法を紹介します。

# sphinx

```
$ pip install sphinx
```


まずは以下のコマンドで`sphinx`のプロジェクト作成します。
（参考にした他の方のは英語が多かったのですが、私の環境では日本語で表示されました）

プロジェクトの言語は`en`でも問題なく日本語の表示ができます。

```
$ sphinx-quickstart docs

> ソースディレクトリとビルドディレクトリを分ける（y / n） [n]: {ENTER}
...
> プロジェクト名: <project_name>
> 著者名（複数可）: <your_name>
> プロジェクトのリリース []: {ENTER}
...
> プロジェクトの言語 [en]: {ENTER}
...
終了：初期ディレクトリ構造が作成されました。

$ cd docs
$ ls
Makefile   _build     _static    _templates conf.py    index.rst  make.bat

```

`sphinx`の初期化を行ったら、`_build`フォルダは gitignore してもらって大丈夫です。

```txt::gitignore
# Sphinx documentation
docs/_build/
```

## sphinxの設定追加

```python:docs/conf.py

- # import os
- # import sys
- # sys.path.insert(0, os.path.abspath('.'))

+ import os
+ import sys
+ import <your_package_root>


- extensions = [
- ]

+ extensions = ['sphinx.ext.autodoc', 'sphinx.ext.napoleon']

- html_theme = 'alabaster'
+ html_theme = 'default'

+ # The master toctree document.
+ master_doc = "index"


```


## Makefileの作成

`Makefile`は自動生成されたものもありますが、以下のように修正すると良いです。


```
SPHINXOPTS    ?=
SPHINXBUILD   ?= sphinx-build
SOURCEDIR     = .
BUILDDIR      = _build

help:
 	@echo " == simple make == "
	@echo "type 'make clean html' to build documentation file"
	@echo ""
	@echo " == command references == "
	@echo "clean: clean build directory"
	@echo "html: build html file"

.PHONY: help Makefile


clean:
	-rm -rf $(BUILDDIR)/*

html:
	sphinx-build $(SOURCEDIR) ./$(BUILDDIR)
	@echo "finish"

```

この設定をして、`Make`とタイプすると`help:`の内容が表示されます。

```
$ make
 == simple make == 
type 'make clean html' to build documentation file

 == command references == 
clean: clean build directory
html: build html file
```

## sphinxを利用したbuild

`./docs`にて、以下のコマンドを実行すると html ファイルが`./_build`に生成されます。

```
$ make clean html
```

# Read The Docs

# 終わりに
