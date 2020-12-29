# 1. はじめに

この記事は、[前回の備忘録](https://qiita.com/gsy0911/items/61198607476ac686ce6f)にて書けなかった事柄について
書いていきます。

項目は以下の通りです。
また、この記事は随時更新予定です。

| 節 | タイトル | 内容 |
|:--:|:--|:--|
| 2 | API Manaegment と統合 | AzureFunctions をドメインを付与した URL で実行可能にする |
| 3 | Azure Identity の利用 | 他のリソースへのアクセスを許可する |

# 2. API Managementと統合

この説では、API Management を利用して、作成した AzureFunctions の`HttpTrigger`の利便性を高くします。

## 2.1 Azure FunctionsのHttpTriggerの不十分な点

API Management を作成する前に、 AzureFunctions の`HttpTrigger`では不十分な点について説明します。
それは、AzureFunctions で発行される URL にあります。

Azure Functions の`Get function URL`から取得した URL を以下に示します。

![スクリーンショット 2020-12-28 1.09.29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/059f8882-3e02-16d9-7fc1-02daff221258.png)

以下に示す URL は**AzureFunctions の名前がドメインの一部になっており**、
AzureFunctions を別の名前に変更する必要に迫られた場合、
利用している箇所の全ての URL を変更する必要があります。
自分が対応可能な範囲ならば問題は小さいかもですが、
この URL を展開していて他の開発者にも変更を迫るのは仕様上あまり良くないです。

加えて、実行を制限するキーとなる`code=***`の`***`の部分は同じ FunctionApp 内の AzureFunctions ごとに異なり、
管理する上でもコスト高になります。

```text:取得したURL
https://gsy0911-function.azurewebsites.net/api/tech/qiita?code=***
```


## 2.2 API Managementの作成

AzureFunctions の不十分な点は以下の２点でした。

1. AzureFunctions の名前がドメインの一部になっており、変更に弱い
2. `code`が AzureFunctions ごとに異なり管理のコストが高い

これらの問題点を解決するために、`API Management`を作成・利用します。

![スクリーンショット 2020-12-28 1.23.23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/2df56d0b-7a11-c476-ad21-801a8caa60a1.png)

`Basics`にて設定するのは名前などの項目です。
`Pricing tier`は`Developer`を設定していますが、それでも月額`5000円程度`は勝手にかかるので
利用していない場合は削除するようにするのが良いです。

![スクリーンショット 2020-12-28 1.24.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/c6e57897-9d6e-dcc7-02c5-8468fb6dda79.png)

`Monitoring`は Application Insights へログを出力するかどうかの設定で、 On にします。

![スクリーンショット 2020-12-28 1.24.57.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/1d045ef4-58fd-586b-8070-88101b5c20f8.png)

`Managed identity`の項目も On にします。

![スクリーンショット 2020-12-28 1.25.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/e340067e-b795-3714-c9a1-bcab187ebb82.png)


それ以外の項目は、デフォルトでいきます。
最後までいったら`Create`ボタンを押下します。
作成の完了まで、地味に時間がかかります。

## 2.3 AzureFunctionsの統合


API Management が作成されたら、早速 Function App の登録をします。
`APIs`・`Function App`の順に押下します。

![スクリーンショット 2020-12-28 2.06.34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/7a5b4e7a-9e65-9e12-bb8d-77e0d97bc527.png)

すると、下のようなポップアップが出てくるので、`Browse`を押下します。

![スクリーンショット 2020-12-28 2.07.33.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/42ab2d42-2fdc-5d9c-9947-e466777226f9.png)

一見わかりにくいですが、`Configure required settings`のボタンを押下します。
すると、右側から新しいタブが出てくるので、追加したい AzureFunctions が含まれる FunctionApp を選択します。

![スクリーンショット 2020-12-28 2.07.41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/d22cc8c0-9a5c-0610-eb1c-a29ae4c6cce1.png)

Function App の名前に`gsy0911-function`が、そして、 AzureFunction に`http_trigger`の項目が追加されています。
さらに、`HTTP methods`には`POST`が、`URL template`には`tech/qiita`があります。
いずれの項目も[前回の記事](https://qiita.com/gsy0911/items/61198607476ac686ce6f)にて設定した項目です。

ここまでできたら、画面左下（下の写真には写ってないです）の`Select`を押下します。

![スクリーンショット 2020-12-28 2.08.06.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/082ff0c9-c895-badf-de00-398abdc15f3e.png)


すると、選択した Functions App が設定されていることがわかります。
ここで、以下の内容は変更できます。

| 項目 | 値 |
|:---:|:---:|
| Display name | API Management にて表示される値 |
| Name | 特に意味はなさそう |
| API URL suffix | **重要**:Base URL の後ろに`/`から付与される値 |


特に気にならなければデフォルトのままで大丈夫ですが、
`API URL suffix`は重要なので慎重に考えた方が良いです。
写真のように`gsy0911-function`のままだと付与される URL は`https://apim-gsy0911.azure-api.net/gsy0911-function/tech/qiita`になります。
空文字列も指定できるため、普段使いの場合は何も入力しない方が良さそうです。
何も入力しない場合は、`https://apim-gsy0911.azure-api.net/tech/qiita`になります。

![スクリーンショット 2020-12-28 2.08.32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/7cc38d49-7a87-adc3-7f24-092a91f19d62.png)

こうして無事に Function App と API Management を統合できました。
`Test`タブを押下し、`http_trigger`を選択すると、右画面の下の方に API の呼び方と、 Key が発行されます。

![スクリーンショット 2020-12-28 2.09.21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/8e14a07e-e4d7-d280-6b85-84694a8803e0.png)

Python からこの API の呼び方は以下の通りです。

```python
import requests

URL = "https://apim-gsy0911.azure-api.net/gsy0911-function/tech/qiita"
SUBSCRIPTION_KEY = "****"

payload = {
    "name": "taro"
}

headers = {
    "Ocp-Apim-Subscription-Key": SUBSCRIPTION_KEY
}

response = requests.post(URL, json=payload, headers=headers)
print(response)
```

`curl`を利用する場合は次のようになります。

```shell
$ curl -X POST -H "Content-Type:application/json" -H "Ocp-Apim-Subscription-Key:****" -d '{"name": "taro"}' https://apim-gsy0911.azure-api.net/gsy0911-function/tech/qiita
```

## 2.4 API Keyの発行

上記の 2.3 でほぼ完了しているのですが、
作成した API を展開する場合は、API Key を別に発行した方が良いです。

理由は以下の通りです。

* 作成した API Key ごとにアクセス制限などがかけられる
* 万が API Key が一流出してしまった場合でも、その API Key を削除すれば被害を最小限に抑えられる

早速、API Key を作成していきます。

`Products`から`Add`を押下します。

![スクリーンショット 2020-12-28 2.30.38.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/c76ac47f-68ea-215c-1375-c626b5cc397a.png)

すると、新しい API Key を作成する画面に行くので
`Display Name`, `Id`, `Published`, `APIs`の項目を設定し、`Create`を押下します。

![スクリーンショット 2020-12-28 2.31.29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/76d2137c-2b31-dc1d-4ee1-47d61863c723.png)

API Key の作成後、`APIs`, `Test`, `http_trigger`の順に追っていき、
`Product`のところにさっき作成した`Display Name`があれば作成できています。

![スクリーンショット 2020-12-28 2.32.00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/7cf1352b-26a0-b146-1def-99327eb14400.png)

上記の手順を踏むことで、本 API を展開する準備ができました。

# 3. Azure Identityの利用

（まだ書いていません）

# 4. おわりに

気がついたことがあれば随時こちらに更新していきます。
