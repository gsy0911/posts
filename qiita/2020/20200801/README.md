# 1. はじめに

本記事は、拙作[【AWS・Python】Slash Commandsを拡張性高くより楽に実装する](https://qiita.com/gsy0911/items/1d95f2ab81f8ae228510)の実装をより楽に修正した物です。

できる物などは変わらずに、よりコード的にすっきりしたのと、実装の際のコストが減るので記事にしました。

# 2. 実装

## 2.1 以前の方法（Chain Of Responsibility）

以前、SlashCommand を実装したときは、GoF のデザインパターンの１つである`Chain Of Responsibility`を利用しました。
詳細は、[前記事](https://qiita.com/gsy0911/items/1d95f2ab81f8ae228510)に譲りますが、実装する際は以下の４つの手順を踏む必要がありました。

1. `CommandExecutor`クラスを継承するクラスを新規作成する
2. `__init__(self)`にて、自身が受け付ける`slash_command`文字列を付与
3. `execute()`関数を実装する
4. 新規作成したクラスを、数珠つなぎに呼べるように`CommandExecutorRequestHandler`クラスを作成する

この方法には、追加が楽というメリットはありますが以下に挙げるようなデメリットもありました。

* `ChainOfResponsibility`の概念を覚える必要がある
* SlashCommands を受け入れるコードを書くまでに４ステップ踏む必要がある
* `CommandExecutorRequestHandler`に新規作成したクラスを追加しないと動作しない
* `CommandExecutorRequestHandler`に追加したクラスの順序によっては予期せぬ動作をする
  * `AllAccept`などの全ての処理を行うクラスを一番最初に追加すると、後続の処理は全て動かなくなる、など
* `ChainOfResponsibility`の性質上、条件分岐を複数回行うため実行時間が遅くなる

## 2.2 今回の方法（Decorator）

以前に対して、今回は`Decorator`というのを利用します。
こちらも、GoF のデザインパターンで登場する名前と同じなのですが、
実際の処理はデザインパターンで登場するような処理とは異なります。

この方法を利用すると、上記のデメリットはほぼ全て解決されます。


### 2.2.1 コマンドを追加する処理

実際にコーディングが必要なのは以下の通りです。
コメントで、上記の手順と対応する番号を付与します。


```python:example.py
@SlashCommand.add(command="/hoge")  # 1, 2: commandがSlashCommandで受け付ける文字列
def hoge(params: dict):  # 3: 処理関数の実装
    """
    /hogeというコマンドを受け取って処理を行う
    """
    return {
        "response_type": "in_channel",
        "text": f"hoge: {str(params)}"
    }


@SlashCommand.add(command="/fuga")  # 1, 2: commandがSlashCommandで受け付ける文字列
def fuga(params: dict):  # 3: 処理関数の実装
    """
    /fugaというコマンドを受け取って処理を行う
    """
    return {
        "response_type": "in_channel",
        "text": f"fuga: {str(params)}"
    }


# 4: 正確には4と対応しないが、数珠つなぎの部分と対応
SlashCommand.execute(params=<payload_from_slack>)
```

### 2.2.2 SlashCommand本体の実装

以下は、以前の実装方法でいうところの`CommandExecutor`クラスです。
decorator で受け取った関数を、`{command: 関数}`という dict で保持しておき、
`SlashCommand.execute()`が呼ばれた際に辞書の文字列でアクセスしています。


```python:slash_command.py
class SlashCommand:
    """
    Decoratorを利用して、処理の登録と実行を行っている
    メッセージの読み取りを始め、処理の移譲をおこなう。
    """
    executor_dict: dict = {}
    guard = None

    def __init__(self):
        pass

    @staticmethod
    def _get_command_from(params: dict) -> str:
        """
        paramsに入ったcommand文字列を取得
        """
        if 'command' not in params:
            raise Exception
        if len(params['command']) == 0:
            raise Exception
        return params['command'][0]

    @classmethod
    def add(cls, command: str, guard=False):
        """
        このdecorator()が受け取るfを登録しておく
        """
        def decorator(f):
            cls.executor_dict[command] = f
            if guard:
                cls.guard = f
            return f

        return decorator

    @classmethod
    def execute(cls, params: dict):
        command = cls._get_command_from(params=params)
        if command in cls.executor_dict:
            return cls.executor_dict[command](params=params)
        else:
            if cls.guard is not None and callable(cls.guard):
                return cls.guard()
            else:
                raise NotImplementedError

```


### 2.3 Chaliceを利用した実装例

最後に`chalice`を利用した実装例を載せます。
デコレータを利用することで、すっきりとした見通しの良いコードになった気がします。

以前の実装方法のソースコードは[ここ](https://github.com/gsy0911/ChatAppForQiita/blob/master/src/lambda/ChatAppForQiita/app.py)にあります。
（実装の都合上、`/hoge`や`/fuga`は`chalicelib`というパッケージの中に含まれています）


```python:app.py

import urllib.parse
from chalice import Chalice

app = Chalice(app_name='<your_project_name>')

# ==================
# このあたりに上記のコード
# ==================
class SlashCommand:
    # 行数の関係で省略、上記と同じコードが入ります。
    pass


@SlashCommand.add(command="/hoge")  # 1, 2: commandがSlashCommandで受け付ける文字列
def accept_hoge(params: dict):  # 3: 処理関数の実装
    """
    /hogeというコマンドを受け取って処理を行う
    """
    return {
        "response_type": "in_channel",
        "text": f"hoge: {str(params)}"
    }


@SlashCommand.add(command="/fuga")  # 1, 2: commandがSlashCommandで受け付ける文字列
def accept_fuga(params: dict):  # 3: 処理関数の実装
    """
    /fugaというコマンドを受け取って処理を行う
    """
    return {
        "response_type": "in_channel",
        "text": f"fuga: {str(params)}"
    }


@app.route('/slack/slash_commands', methods=['POST'], content_types=['application/x-www-form-urlencoded'])
def receive_slash_command():
    request = app.current_request
    if request.raw_body is None:
        # 予期しない呼び出し。400 Bad Requestを返す
        return {'statusCode': 400}
    payload = urllib.parse.parse_qs(request.raw_body.decode('utf-8'))

    return SlashCommand.execute(params=payload)
```


# 3. おわりに

以前までは、`ChainOfResponsibility`ってかなり便利だなと思っていたのですが、
実装の手順が煩雑になったりとデメリットの方が最近は気になりました。

その点、decorator は、メタプログラミング、と名のつく通り、かなりいろいろなことができるので便利です。
（あまりに複雑なものは、後々の自分や他の人への可読性という意味では危ないですが）

この方法を利用すれば、SlashCommand の処理のみに集中できるので良いかと思います。
