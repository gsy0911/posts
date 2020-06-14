# はじめに

そろそろ`Python3.8`がリリースされて、半年以上が経ちます
（最初のリリース日は 2019 年 10 月 14 日（[参考](https://www.python.org/downloads/))です）。
`Python3.8`が使えていません。

[面白そうな新機能](https://qiita.com/ksato9700/items/3846e8db573a07c71c33)がたくさんあるのにです。

と言うことで、`Mac OS`に`Python3.8`を入れていきます。

ただ、巷にあふれている`pyenv`を利用する方法ではなく、
`homebrew`で入れられる`Python@3.8`を利用する方法です。
（`pyenv`アレルギーなので）

# 手順

## 1. `homebrew`にて`python@3.8`をインストール

`homebrew`を入れてない場合は、`homebrew`のインストールから行いましょう
`Python@3.8`のドキュメントは[ここ](https://formulae.brew.sh/formula/python@3.8)にあります。

```zsh
$ homebrew install python@3.8
```

インストール先は`(brew --prefix)/opt/python@3.8/bin/python3`になります。


## 2. `PATH`を通す

`homebrew`で`Python@3.8`がインストールできたら、[ここ](https://stackoverflow.com/questions/60453261/how-to-default-python3-8-on-my-mac-using-homebrew)を参考に`PATH`を通します。

`Catalina`からデフォルトのシェルは`zsh`なので、`~/.zshrc`に以下を追記します。
私の環境では、`(brew --prefix)`は`/usr/local`なので、以下のパスになります。

```zsh:.zshrc
# pathを通す
export PATH="/usr/local/opt/python@3.8/bin:$PATH"
```

```zsh
# ターミナルの再起動
$ zsh -l
# パスが通っているか確認
$ which python3.8
/usr/local/opt/python@3.8/bin/python3.8
```

## 3. poetryのenv変更

最後に、`poetry`の利用する`Python`バージョンの変更です。
[ここ](https://github.com/python-poetry/poetry/issues/1735)を参考に
`poetry`を作成した環境直下にて以下コマンドを打ちます。

```zsh:.../someproject/
# 環境の切り替え
$ poetry env use 3.8

# packageのupdateしないとimport errorになります
$ poetry update

# shellの起動
$ poetry shell

# python shellの起動
$ python
Python 3.8.2 (default, Mar 11 2020, 00:29:50)
[Clang 11.0.0 (clang-1100.0.33.17)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```


# 終わりに

と言うことで、念願の`Python3.8`を使えるようになりました。
皆さんも良き`Poetry` + `Python3.8`ライフを。
