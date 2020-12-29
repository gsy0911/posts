# はじめに

この記事では、AzureFunctionsでよく私が利用している`HttpTrigger`と`TimerTrigger`を
デプロイして動かすところまでを解説します。

結構長くなったので、目次を付与しておきます。

1. 前準備
    1. ローカル環境：AzureFunctionsをデプロイするのに必要なソフトのインストール
    1. Azureのクラウド環境：AzureFunctionsをデプロイするのに必要なサービスを事前に作成  
1. AzureFunctionsの種類
1. AzureFunctionsの実装
    1. HttpTrigger
    1. TimerTrigger
1. AzureFunctionsのデプロイ


# 前準備

AzureFunctionsをdeployするために、ローカル環境とAzureのクラウド環境側で以下のような準備が必要です。

## ローカル環境

本記事ではMacで環境構築などを行います。

AzureFunctionsをデプロイするために、以下の2つのソフトのインストールが必要です。

* [Azure CLI](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli?view=azure-cli-latest)
* [Azure Function Core Tools](https://docs.microsoft.com/ja-jp/azure/azure-functions/functions-run-local?tabs=macos%2Cpython%2Cbash)

Macへのインストールは`brew`を利用すれば簡単にできます。

```shell
# Azure CLIのインストール
$ brew update && brew install azure-cli

# Azure Function Core Toolsのインストール
$ brew tap azure/functions
$ brew install azure-functions-core-tools@3

# AzureにAzureCLIを利用してログイン
$ az login
```

今回デプロイするに当たって利用しているバージョンは次の通りです。

```shell
# Azure CLIのバージョン
$ az --version
azure-cli                         2.16.0
...

# Azure Functions Core Toolsのバージョン
$ func --version
3.0.2912
```

## Azureのクラウド環境

Azureのアカウントはすでにあることを前提にしています。

AzureFunctionsをデプロイするにあたっては、
以下のリソースを前もって準備しておくと便利です。

| サービス名 | 用途 | 推奨されるprefix | 必須 |
|:---:|:---:|:---:|:---:|
| Resource Group | AzureFunctionsのデプロイに必要なサービスをまとめる | `rg-` | O |
| StorageAccount | ファイルを保持するストレージ | `st-` | O |
| Log Analytics | ログを保持する | `log-` | O |
| Application Insights | ログを表示する | `appi` | O |
| KeyVault | シークレットなどのコードに直接載せたくない値を保持する | `kv-` | - |
| AzureIdentity | AzureFunctions外のリソースにアクセスする権限などを管理する | - | - |
| Api Management | 外部のパートナーや社内の開発者にAPIを公開する | `apim-` | - |

これらのリソースは、基本的には**一度作成した後に作成する必要はありません**。
ただ、今回はAzureFunctionsをコンソールから作成していきます。
すでに、上記の必須なサービスがある場合は、次節まで読み飛ばしてもらって大丈夫です。

また、これらのサービスの名前のprefixは、Azureが公式に推薦する形式があります。
そのため、この[公式ドキュメント](https://docs.microsoft.com/ja-jp/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations)をみておくことをお勧めします。
（今回デプロイしたサービスは一部その例に従ってはいません）

### Resource Group

### Storage Account

### Log Analytics + Application Insights

### Key Vault

### AzureIdentity

# AzureFunctionsの種類

AzureFunctionsは、様々な起動方法（`トリガー`）があります。
特に私が利用することが多いのは次の4つです。

* HttpTrigger
* TimerTrigger
* QueueTrigger
* BlobTrigger

そのほかのトリガーの例に関しては、[公式ドキュメント](https://docs.microsoft.com/ja-jp/azure/azure-functions/functions-triggers-bindings?tabs=csharp)を確認してください。
また、今回は特に利用しないので説明しませんが、`トリガー`の他に`バインディング`という概念もあります。
詳しくは、上記の公式ドキュメントに解説されていますので読んでください。

# AzureFunctionsの実装

初めから作成する場合は以下のコマンドを実行すると、必要なファイルを生成してくれます。
実行するruntimeは今回はPythonなので3を入力します。

```shell

$ func init test_function
Select a number for worker runtime:
1. dotnet
2. node
3. python
4. powershell
5. custom
Choose option: 3 {<- type `3`}
python
Found Python version 3.8.6 (python3).
Writing requirements.txt
Writing .gitignore
Writing host.json
Writing local.settings.json
Writing {current_directory}/test_function/.vscode/extensions.json

# 作成後に関数の名前のフォルダが作成されるのでそこに移動
$ cd test_function

# func initコマンドで作成されたファイル
$ tree
.
├── host.json
├── local.settings.json
└── requirements.txt
```

これにて、AzureFunctionsにデプロイするのに必要なファイルが生成されました。


## `requirements.txt`の修正

バリデーション用に`cerberus`というライブラリを追加しておきます。

```diff:requirements.txt
# Do not include azure-functions-worker as it may conflict with the Azure Functions platform

azure-functions
+ cerberus
```

## HttpTriggerの作成

はじめにHttpTriggerを作成します。

```shell
# test_function/以下にて実行
$ func new
Select a number for template:
1. Azure Blob Storage trigger
2. Azure Cosmos DB trigger
3. Azure Event Grid trigger
4. Azure Event Hub trigger
5. HTTP trigger
6. Azure Queue Storage trigger
7. RabbitMQ trigger
8. Azure Service Bus Queue trigger
9. Azure Service Bus Topic trigger
10. Timer trigger
Choose option: 5 {<- type `5`}
HTTP trigger
Function name: [HttpTrigger] test_http_trigger {<- type `test_http_trigger`}
Writing {current_directory}/test_http_trigger/__init__.py
Writing {current_directory}/test_http_trigger/function.json
The function "test_http_trigger" was created successfully from the "HTTP trigger" template.

# HttpTriggerの関数を作成したあとのファイル構成
$ tree
.
├── host.json
├── local.settings.json
├── requirements.txt
└── test_http_trigger
    ├── __init__.py
    └── function.json
```

### HttpTriggerの`functions.json`

ここの`name`は、`__init__.py`の`main()`関数の引数の名前と一致させる必要があります。

```json:function.json

{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
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
    }
  ]
}
```

今回は`POST`メソッドを受け取るAzureFunctionsを作成したいので、`get`を削除します。
また、API Managementと統合した際にURLのrouteを設定したいので、`tech/qiita`を追記します。

```diff:function.json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
+     "route": "tech/qiita",
      "methods": [
-        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}

```

### HttpTriggerの`__init__.py`

デフォルトで作成されるのは以下のテンプレートですが、
* `logging`モジュールをそのまま利用していたり、
* `HttpResponse`の返り値の長さが120文字を超えていたり（`PEP8`違反）
* ValueErrorの例外処理を`pass`していたり
お世辞にも参考にして良いサンプルプログラムではないです。

```python:修正前の__init__.py
import logging

import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello, {name}. This HTTP triggered function executed successfully.")
    else:
        return func.HttpResponse(
             "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.",
             status_code=200
        )

```

そのため、全て書き換えます。
ここで、先ほどrequirementsに追記しておいたcerberusというライブラリを利用して入力パラメータのバリデーションを行なっています。

```python:修正後の__init__.py

import json
from logging import getLogger, INFO

import azure.functions as func
from cerberus import Validator

logger = getLogger()
logger.setLevel(INFO)


# Validation
SCHEMA = {
    "name": {"type": "string"}
}


def main(req: func.HttpRequest) -> func.HttpResponse:
    """
    以下のようなpayloadを受け取る関数
    {
        "name": "taro",
        "age": 10
    }
    """
    # POSTのみを対象としているため
    payload = req.get_json()

    # バリデータを作成
    validator = Validator(SCHEMA)

    # バリデーションがうまく通らない場合エラーを吐く
    # 不明なパラメータの入力も`allow_unknown`で許可している
    if not validator.validate(payload, allow_unknown=True):
        return func.HttpResponse(
            body=json.dumps(validator.errors),
            mimetype="application/json",
            charset="utf-8",
            status_code=400
        )

    # 正しい入力があった場合は、受け取ったpayloadをそのまま返す
    return func.HttpResponse(
        body=json.dumps(payload),
        mimetype="application/json",
        charset="utf-8",
        status_code=200
    )

```

これにてHttpTriggerのコードはできました。

## TimerTriggerの作成

次にTimerTriggerを作成します。
先ほどと同様にコマンドからサンプルのプログラムを作成します。

```shell
$ func new
Select a number for template:
1. Azure Blob Storage trigger
2. Azure Cosmos DB trigger
3. Azure Event Grid trigger
4. Azure Event Hub trigger
5. HTTP trigger
6. Azure Queue Storage trigger
7. RabbitMQ trigger
8. Azure Service Bus Queue trigger
9. Azure Service Bus Topic trigger
10. Timer trigger
Choose option: 10 {<- type `10`}
Timer trigger
Function name: [TimerTrigger] test_timer_trigger {<- type `test_timer_trigger`}
Writing {current_directory}/test_timer_trigger/readme.md
Writing {current_directory}/test_timer_trigger/__init__.py
Writing {current_directory}/test_timer_trigger/function.json
The function "test_timer_trigger" was created successfully from the "Timer trigger" template.

# 作成されたファイルの確認
$ tree
.
├── host.json
├── local.settings.json
├── requirements.txt
├── test_http_trigger
│   ├── __init__.py
│   └── function.json
└── test_timer_trigger
    ├── __init__.py
    ├── function.json
    └── readme.md
```

### TimerTriggerの`functions.json`

実行タイミングは`schedule`項目にて修正できます。
この例だと5分ごとに実行されるようになっています。

```json:functions.json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "mytimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 */5 * * * *"
    }
  ]
}
```

### TimerTriggerの`__init__.py`

```python:修正前の__init__.py
import datetime
import logging

import azure.functions as func


def main(mytimer: func.TimerRequest) -> None:
    utc_timestamp = datetime.datetime.utcnow().replace(
        tzinfo=datetime.timezone.utc).isoformat()

    if mytimer.past_due:
        logging.info('The timer is past due!')

    logging.info('Python timer trigger function ran at %s', utc_timestamp)

```


# AzureFunctionsのデプロイ

デプロイには`Makefile`を利用すると便利です。

# 終わりに
