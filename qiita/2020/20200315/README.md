# はじめに
本記事は、AzureFunctionsのうちQueue Storageのバインドを利用してAzureFunctionsを連鎖させることを目指します。

概要としては以下の通りで、

* HTTPリクエストで起動したAzureFunctionsがQueueStorageにメッセージをenqueue
* Queueへのenqueueをトリガーに別のAzureFunctionsを起動する

図で表すとこんな感じです。

![azure-Page-2 (1).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/652bcf3b-90f4-d274-bd5d-27e04ea1e5ef.png)

1. Userなどがcurlを叩く
2. ②のAzureFunctionが起動
3. Queue Storageにメッセージがenqueue
4. enqueueをtriggerに③のAzureFunctionsが起動

# 前提

* VSCodeなどを利用してAzureFunctionsの作成・デプロイができる
* ②と③のAzureFunctionsは同一リージョン・Storage Accountに紐づいている

# 準備・実装

## 準備：①Queue Storage

Queue Storageに「＋Queue」ボタンから`piyo`というQueueを作ります。

![storage_account_qiita.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/39b12259-9972-5e2e-daeb-788bb0be3cfe.jpeg)

## 実装：②Azure Functions

VSCodeからHTTP TriggerのAzure Functionsを作成し、自動生成されたコードに対して修正していきます。
追加するのは下部の5項目、`"type", "direction", "name", "queueName", "connection"`です。
（VSCodeならば、add bindingでも追加できます。）

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
この`name`がpythonプログラム上で受け取る引数名になります。
（最初はここがわからなくて、関数の実行ログも出力が遅いのでエラーの原因の特定に時間がかかりました。。。）

| 項目名 | 定数 | パラメータ | 説明 |
|:--|:--:|:--|:--|
| type | 〇 | queue | bindingの種類 |
| direction | 〇 | out | bindingの方向。出力のためout |
| name |  | fuga1 | pythonの実行関数で受け取る際の**引数の名前** |
| queueName |  | piyo | 上記のステップで作成したQueueの名前 |
| connection | 〇 | AzureWebJobsStorage | 今回は同一のStorage Accountのため定数。通常は接続文字列などを入れる？ |


次に、Pythonのコード本体です。
ここで、main関数の引数に`fuga1`を入れます。すなわち、`function.json`と同じ名前を指定します。
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

## 実装：③Azure Functions

②のときと同様にVSCodeから、今度はQueue Triggerの関数を自動生成します。
今回は入力のbindingのみです。
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

次に、Pythonのコード本体です。
main関数の引数のみ`function.json`で指定した変数名`fuga2`に修正します。

```python:__init__.py
import logging
import azure.functions as func


def main(fuga2: func.QueueMessage) -> None:
    logging.info('Python queue trigger function processed a queue item: %s',
                 fuga2.get_body().decode('utf-8'))
```

# 実行例

ローカルのcurlから叩くと、動いていることがログからわかります。

まずは、HTTPリクエストで②のAzureFunctionsが起動し、
![test_function.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/18384385-6343-e3f4-680a-059371da646b.jpeg)

③のAzureFunctionsがQueueをトリガーにして起動しています。
![queue_trigger.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/d8d3bc44-ea9f-303e-fe01-ae603d5ad9c4.jpeg)

Queueにメッセージが入ってから2秒後にAzureFunctionsが動いているのがわかりますね。
（関数名には目を瞑ってください。。。）

# おわりに

AzureFunctionsのQueueTriggerで連鎖できることが分かりました。
実際に動くサービスを作成する場合はQueueの名前を上手に生成したり、複数個のout bindingsを利用したりすることでAzureFunctionsをいい感じに連鎖できそうですね。

# 参考
* [[Azure Functions] Queue StorageバインドでFunctionから別Functionを呼び出す](https://qiita.com/momotaro98/items/35fb835ce92e5d5bacb2)
* [【公式】Azure Functions における Azure Queue storage の出力バインド](https://docs.microsoft.com/ja-jp/azure/azure-functions/functions-bindings-storage-queue-output?tabs=python)
