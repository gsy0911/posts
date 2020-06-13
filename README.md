# posts

The repository stores my writings and pics posted to

* Qiita
* GitHub Gist
* somewhere in private

## Contents

* Qiita
* GitHub Gist

## textlint

この[Qiita](https://qiita.com/takasp/items/22f7f72b691fda30aea2)を参考にして、VSCode に文章の校正環境を構築します。

1: install npm modules

```
# init
$ npm init -y

# install dependencies
$ npm install --save-dev \
    textlint \
    textlint-rule-preset-ja-spacing \
    textlint-rule-preset-ja-technical-writing \
    textlint-rule-spellcheck-tech-word

# 設定ファイルの作成
# npx textlint --init
```

2: install [VS Code Extention](https://marketplace.visualstudio.com/items?itemName=taichi.vscode-textlint)