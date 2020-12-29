# 1. はじめに

本記事では、Slack へのファイルアップロードをトリガーとして、
ファイルの受け取りと何かしらの処理をして Slack へ送り返す処理についてまとめます。

環境は AWS Lambda と Python を利用します。

# 2. 実装

## 2.1 できるもの

ファイルをアップロードすると、
アップロードしたユーザにメンションして、何かしらの処理を加えたファイルを返します。

![スクリーンショット 2020-07-25 17.11.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/6d47fcc4-10e1-bc52-b2a1-fec8c34df593.png)

## 2.2 処理のイメージ図

Slack → AWS → Slack へのデータの流れです。

![_qiita_slack_event_api.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/157155bb-c097-047d-7225-2aad6e54cc08.png)

1. Slack 上でファイルがアップロードされると、API Gateway に設定したエンドポイントが呼ばれる
2. API Gateway が Lambda を起動
3. 処理用の Lambda をさらに起動し、
4. 最初の Lambda は、処理を返す
5. 最後に呼ばれた Lambda がファイルを処理してメッセージとともに Slack へアップロード

`3`にて別の Lambda を起動するのは、Slack へのレスポンスを３秒以内に返す必要があるためです[参考](https://qiita.com/kanaxx/items/c29267d88c3fb2cc381c#%E3%81%8A%E4%BD%9C%E6%B3%95)

## 2.3 slack上での準備

Slack の App を管理しているページの`Event Subscription`の項目を有効にしておきましょう。

![スクリーンショット 2020-07-25 17.22.16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/851e67af-1d70-9d1c-21a2-5c60c18122ce.png)

加えて、`Incoming Webhooks`の項目も有効にしておいてください。

## 2.4 Chaliceでの実装

`Chalice`とは、`AWS Lambda`やそれに付随するサービスを簡単に構築してくれる AWS 公式のライブラリです。

### 2.4.1 SlackからのEventを受け取るLambda(API Gateway + Lambda)の実装

まずは、Slack からの`Event`を受け取る Lambda を実装します。
処理の内容は以下の通りです。

1. Slack からの`payload`を受け取る
2. BOT からの送信でなければ、ファイルを処理する Lambda を起動
3. 最後に、Slack から受け取った`payload`を返す

`3`をしている理由としては、Slack から送信された`payload`に含まれる`challenge`パラメータを返す必要があるためです。
これをしていない場合は、2.3 項で設定した URL が`Verified`になりません。

```python:app.py
import io
import json
import logging
import os
import requests
from slack import WebClient
from slack.errors import SlackApiError
import boto3

app = Chalice(app_name='<your chalice-app name>')
logger = logging.getLogger()
logger.setLevel(logging.INFO)
# lambda client to invoke
lambda_client = boto3.client("lambda")

BOT_USER_ID = "<appのuser_id>"

@app.route('/your/root', methods=['POST'])
def event_subscription():
    request = app.current_request
    if request.raw_body is None:
        # 予期しない呼び出し。400 Bad Requestを返す
        return {'statusCode': 400}
    payload = request.json_body
    logger.info(f"payload:= {payload}")
    user_id = payload['event']['user_id']
    # BOTからファイルを上た場合は、Lambdaでの処理をしないようにする
    # ループを避けるため
    if user_id == BOT_USER_ID:
        return payload

    event_handler_lambda = "<この次に実装するLambdaのARN>"
    lambda_client.invoke(
        FunctionName=event_handler_lambda,
        InvocationType='Event',
        Payload=json.dumps(payload)
    )

    return payload
```

### 2.4.2 API ファイルを処理するLambdaの実装

次は、アップロードされたファイルに対して処理をする Lambda を実装します。
Slack へファイルがアップロードされると、`file_id`が発行されます。
その`file_id`からファイルを取得し、処理を行っていきます。

また、その`file_id`にはファイルがアップロードされた`chennel_id`が付与されているので
これを利用してファイルをアップロードするチャンネルを指定します。

ユーザへのメンションは`<@{user_id}>`で行えます。
`user_id`は、ファイルがアップロードされたイベントに付与されているのでそれを取得して行います。

ファイルの処理は、今は適当に行っているので、いい感じに変更してください。
`client.files_upload()`の関数へ渡せる引数として`file`と`content`の２種類があります。
違いは以下の通りです。

* file: ファイル名を指定して、アップロードする。例：`file="test.csv"`
* content: bytes オブジェクトをアップロードする。例：`content=json.dumps({"aaa": "bbb"}).encode('utf-8')`

どちらでも行けるので便利な方を利用してください。
ちなみに、Lambda だと`/tmp`以下の領域は`500MB`くらいまで自由に読み書きができるので、そこを利用したら良いかと思います。

```python:app.py
# =============================== #
# ここには2.4.1項で実装した内容がある想定 #
# =============================== #

@app.lambda_function()
def event_handler(event, _):
    # globalで読み込むとslackへのレスポンスに3秒以上かかるため、ここでpandasを読み込む
    import pandas as pd
    # tokenとslack_clientの生成
    slack_oauth_token = os.environ['OAuthAccessToken']
    slack_bot_token = os.environ['BotUserOAuthAccessToken']
    oauth_client = WebClient(token=slack_oauth_token)
    bot_client = WebClient(token=slack_bot_token)

    # call file info to get url
    file_id = event['event']['file_id']
    logger.info(f"file_id:= {file_id}")
    response = oauth_client.files_info(file=file_id)
    file_name = response.data["file"]["name"]
    file_url = response.data["file"]["url_private"]
    # slack上でファイル共有をした場合は1つのchannel_idしか入らないため
    file_upload_channel_id = response.data['file']['channels'][0]

    user_id = event['event']['user_id']
    logger.info(f"file from {user_id}")
    print("Downloaded " + file_name)

    # download file
    file_data = _get_slack_file_bytes(slack_token=slack_oauth_token, file_url=file_url)
    converted_data = b''
    if file_name.endswith(".json"):
        sent_data = json.loads(file_data)
        sent_data['accepted'] = "hello, world"
        converted_data = json.dumps(sent_data).encode('utf-8')
    elif file_name.endswith(".csv"):
        df = pd.read_csv(io.BytesIO(file_data))
        df['accepted'] = "world"
        converted_data = df.to_csv(index=False).encode('utf-8')

    # response
    try:
        chat_response = bot_client.chat_postMessage(
            channel=file_upload_channel_id,
            text=f"<@{user_id}> {file_name} is accepted :tada:"
        )
        logger.info(f"chat_response:= {chat_response}")

        upload_response = bot_client.files_upload(
            content=converted_data,
            title=f"converted_{file_name}",
            filename=f"converted_{file_name}",
            initial_comment="here is your file",
            channels=file_upload_channel_id
        )
        logger.info(f"upload_response:= {upload_response}")

    except SlackApiError as e:
        # You will get a SlackApiError if "ok" is False
        assert e.response["error"]
        print(e)
        print(e.__str__())
    return {"ok": True}


def _get_slack_file_bytes(slack_token: str, file_url) -> bytes:
    r = requests.get(file_url, headers={'Authorization': f'Bearer {slack_token}'})
    # get binary content
    return r.content

```
## 2.5 LambdaのdeployとSlackへのURL設定

ここまできたら、`chalice`のコマンドで AWS に実装して、発行される URL を取得します。

```zsh
$ chalice deploy
Creating deployment package.
Updating policy for IAM role: <chalice_project_name>-dev
Updating lambda function: <chalice_project_name>-dev-event_handler
Updating lambda function: <chalice_project_name>-dev
Updating rest API
Resources deployed:
  - Lambda ARN: arn:aws:lambda:<region>-<account_number>:function:<chalice_project_name>-dev-event_handler
  - Lambda ARN: arn:aws:lambda:<region>-<account_number>:function:<chalice_project_name>-dev
  - Rest API URL: https://<chalice_generated_chars>.execute-api.<region>.amazonaws.com/api/
```

API Gateway の URL を取得したら、2.3 項での`Event Subscription`の`Request URL`に設定します。
この時、プログラムの`@app.route(/your/root, ...)`に設定した`/your/root`を追記します。
例で、`/your/root`としましたが、`/slack/event`などが良いと思います。

```
https://<chalice_generated_chars>.execute-api.<region>.amazonaws.com/api/your/root
```

## 2.6 実行時の注意

### 2.6.1 Slack Bot Userの実行権限付与

おそらく、初めて`Event Subscription`や`Incoming Webhooks`の項目を利用していると、
Bot の権限が足りずに関数の実行が失敗します。

その場合は、Slack App の以下のページから必要そうな権限を追加していってください。
関数の実行に必要な権限は実行時に失敗したら、エラーメッセージに含まれています。

![スクリーンショット 2020-07-25 17.55.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/5382e7e3-87cb-6079-2abc-de5931529aee.png)

### 2.6.2 LambdaがLambdaを実行する権限を付与

Lambda から Lambda を呼ぶ権限も IAM に付与する必要があります。
AWS コンソールから、Lambda に付与されているロールを選び、「ポリシーをアタッチします」ボタンから
`AWSLambdaFullAccess`を付与してください（権限が強すぎるので本当はよくないのですが）。

![スクリーンショット 2020-07-25 18.10.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/8d9f6432-599a-3a2b-7dcc-5c730844dc7d.png)

### 2.6.3 Slackでファイルを実際にルームに共有される前にLambdaが実行されてしまう。

`file`に関する Slack の`Event Subscription`のうち、`file_upload`を選択してしまうと、
ルームに共有しようとしているファイルが Slack 上のサーバーへアップロードが完了した時点で、Event が発火します。

処理そのものに影響はないのですが、挙動としてちょっと気持ち悪いので`file_public`の Event が発火した場合に
Lambda などの処理を行った方が良いです。


# 3. おわりに

今回は、Slack を利用してファイルの送受信を行えるようにしてみました。
ファイル以外にも Slack の`Event Subscription`には、`スタンプが押されたら`とか`メンションがあったら`とかいろいろあるのでぜひ遊んでみてください。
`Event Subscription`では、イベントのタイプを`{..., 'event': {'type': 'file_public', 'file_id': '', ...}}`と言う
dict の'type'で受け取れるので、[以前書いたChainOfRespontibilityでゴニョゴニョ](https://qiita.com/gsy0911/items/1d95f2ab81f8ae228510)とかも利用できると思います。

Slack が提供している公式ライブラリの[SlackClient](https://github.com/SlackAPI/python-slackclient)が
かなり使いやすく、自分で独自の関数を組む必要がないので便利でした。

また、`Event Subscription`の設定方法などは【[Slackにファイルをアップロードする](https://qiita.com/yaju/items/2e1ab8a25b6e207bfbe6)】が参考になるかもです。

# 参考

* [slack api files.upload](https://api.slack.com/methods/files.upload)
* [mention in slack](https://stackoverflow.com/questions/40771924/how-to-mention-user-in-slack-client)
