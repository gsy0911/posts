# はじめに
本記事は、`AzureFunctions`のうち`Queue Storage`のバインドを利用して`AzureFunctions`を連鎖させることを目指します。

概要としては以下の通りです。

* HTTP リクエストで起動した`AzureFunctions`が`QueueStorage`にメッセージを`enqueue`
* `Queue`への`enqueue`をトリガーに別の`AzureFunctions`を起動する


図示するとこんな感じです。

![azure-Page-2 (1).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/652bcf3b-90f4-d274-bd5d-27e04ea1e5ef.png)

1. User が curl を叩く
2. ②の`AzureFunction`が起動
3. `Queue Storage`にメッセージが`enqueue`
4. `enqueue`を trigger に③の`AzureFunctions`が起動

# 前提

* VSCode などを利用して`AzureFunctions`の作成・デプロイができる
* ②と③の`AzureFunctions`は同一リージョン・同一`Storage Account`に紐づいている

# 準備・実装

## 準備：① `Queue Storage`

`Queue Storage`に「＋Queue」ボタンから`piyo`という`Queue`を作ります。

![storage_account_qiita.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/39b12259-9972-5e2e-daeb-788bb0be3cfe.jpeg)

## 実装：② `AzureFunctions`

VSCode から HTTP Trigger の`AzureFunctions`を作成し、自動生成されたコードに対して修正していきます。
追加するのは下部の５項目、`"type", "direction", "name", "queueName", "connection"`です。
（VSCode ならば、`add binding`でも追加できます）

```json:function.json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "fuga1",
      "queueName": "piyo",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```
以下に簡単に追加項目の説明をします。はまりやすい設定は、`name`です。
この`name`が`Python`プログラム上で受け取る引数名になります。
（最初はここがわからなくて、関数の実行ログも出力が遅いのでエラーの原因の特定に時間がかかりました）

| 項目名 | 定数 | パラメータ | 説明 |
|:--|:--:|:--|:--|
| type | 〇 | queue | `binding`の種類 |
| direction | 〇 | out | `binding`の方向。出力のため`out` |
| name |  | fuga1 | `Python`の実行関数で受け取る際の**引数の名前** |
| queueName |  | piyo | 上記のステップで作成した`Queue`の名前 |
| connection | 〇 | AzureWebJobsStorage | 今回は同一の`Storage Account`のため定数。 |


次に、`Python`のコード本体です。
ここで、main 関数の引数に`fuga1`を入れます。すなわち、`function.json`と同じ名前を指定します。
**注意 `name`に`_`などの記号を入れると動かなくなります。**

```python:__init__.py
import logging
import azure.functions as func


def main(req: func.HttpRequest, fuga1: func.Out[str]) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    # パラメータの取得
    name = req.params.get('name')
    # queueに入れる
    fuga1.set(name)
    
    # 返り値（自動生成の場合はもうちょっと長いけどここは関係ないので省略）
    return func.HttpResponse("\n".join(return_list))

```

## 実装：③ `AzureFunctions`

②のときと同様に VSCode から、今度は`Queue Trigger`の関数を自動生成します。
今回は入力の`binding`のみです。
`queueName`には`Queue Storage`の`piyo`を指定します。
`name`は`fuga2`にしています。この`name`は`AzureFunctions`ごとに固有に決めることができるからです。

```json:function.json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "fuga2",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "piyo",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

次に、`Python`のコード本体です。
main 関数の引数のみ`function.json`で指定した変数名`fuga2`に修正します。

```python:__init__.py
import logging
import azure.functions as func


def main(fuga2: func.QueueMessage) -> None:
    logging.info('Python queue trigger function processed a queue item: %s',
                 fuga2.get_body().decode('utf-8'))
```

# 実行例

ローカルの curl から叩くと、動いていることがログからわかります。

まずは、HTTP リクエストで②の`AzureFunctions`が起動し、
![test_function.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/18384385-6343-e3f4-680a-059371da646b.jpeg)

③の`AzureFunctions`が`Queue`をトリガーにして起動しています。
![queue_trigger.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/d8d3bc44-ea9f-303e-fe01-ae603d5ad9c4.jpeg)

`Queue`にメッセージが入ってから２秒後に`AzureFunctions`が動いているのがわかりますね。
（関数名には目を瞑ってください）

# おわりに

`AzureFunctions`の`QueueTrigger`で連鎖できることが分かりました。
実際に動くサービスを作成する場合は`Queue`の名前を上手に生成したり、複数個の`out bindings`を利用したりすることで`AzureFunctions`をいい感じに連鎖できそうですね。

# 参考
* [[Azure Functions] Queue StorageバインドでFunctionから別Functionを呼び出す](https://qiita.com/momotaro98/items/35fb835ce92e5d5bacb2)
* [【公式】Azure Functions における Azure Queue storage の出力バインド](https://docs.microsoft.com/ja-jp/azure/azure-functions/functions-bindings-storage-queue-output?tabs=python)
