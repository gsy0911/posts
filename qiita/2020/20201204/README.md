# 1. はじめに

日本時間の2020年12月2日午前1時ごろから始まった１つ目の KeyNote で、
突如`LambdaContainer`が発表されました（[参考](https://dev.classmethod.jp/articles/lambda-support-oci-container-image/)）。

私も、この発表はリアルタイムでみていて「うぉぉぉぉ」と心の中で雄叫びを上げました。
コンソールなどではすでに利用できるみたいです。

また、驚いたのが、[AWS CDK](https://github.com/aws/aws-cdk)でも、`LambdaContainer`の発表から数時間後には
バージョンが`1.76.0`に上がり、その際に`LambdaContainer`がサポートされたことです。

# 2. 実装

CDK での実装に関するドキュメントは[公式](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-lambda.DockerImageCode.html)にて提供されており、これまでのデプロイと大差なく利用できそうです。

また、 DockerFile は[Developers.IOさんの記事](https://dev.classmethod.jp/articles/lambda-container-pkgfmt-with-sam-cli/)を参考にしました。

## 2.1 ファイル構成

CDK をデプロイするファイル構成は以下の通りです。

```shell
lambda-container/
├── app.py
├── cdk.json
└── docker/
    ├── Dockerfile
    ├── app.py
    └── requirements.txt
```


## 2.2 Lambdaのメイン関数

特に意味のない Lambda の関数です。
requirements.txt にてインストールしたパッケージが正しく利用できるか、
pandas のバージョンを表示させてみます。

```python:docker/app.py
import pandas as pd

def lambda_handler(event, _):
    print(pd.__version__)
    return event

```

## 2.3 Dockerfile

Lambda に載せる Dockerfile の例は以下の通りです。
（こちらの Dockerfile は参考の記事をそのまま利用させてもらいました）

```Dockerfile
FROM public.ecr.aws/lambda/python:3.8

COPY app.py requirements.txt ./

RUN python3.8 -m pip install -r requirements.txt -t .

# ここのコマンドは
# app.pyのスクリプトにある、lambda_handlerという関数をエントリポイントにする、
# という意味です。
CMD ["app.lambda_handler"]
```

他の Python のバージョンは[ここ](https://docs.aws.amazon.com/lambda/latest/dg/python-image.html#python-image-clients)を参考にしてください。
（`2.7`とかも一応利用できるみたいですね）


## 2.4 CDK上での実装

公式ドキュメントを参考に実装していきます。

```python:app.py
from aws_cdk import (
    aws_lambda as lambda_,
    core,
)


class LambdaContainer(core.Stack):
    def __init__(self, app: core.App, _id: str):
        super().__init__(scope=app, id=_id)

        _ = lambda_.DockerImageFunction(
            scope=self,
            id="container_function",
            code=lambda_.DockerImageCode.from_image_asset(
                directory="docker",
                repository_name="lambda_container_example"
            )
        )

def main():
    app = core.App()
    LambdaContainer(app, "LambdaContainer")
    app.synth()


if __name__ == "__main__":
    main()
```

## 2.5 デプロイ

ここまでできたら、CDK の app.py があるディレクトリにて、以下のコマンドを実行すればデプロイ可能です。

```shell
# pythonコードが正しいか確認
# 構文エラーがなければ、デプロイ可能なStackが表示されます。
$ cdk ls
LambdaContainer

# 実際にdeploy
$ cdk deploy
```

デプロイを開始すると、いつものようにリソースを作成するか聞かれます。
![スクリーンショット 2020-12-03 23.55.11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/fd782b76-8653-ec95-7b42-d32809752b86.png)

`y`を押して進むと、 docker の build が始まり、
![スクリーンショット 2020-12-03 23.56.13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/73d540bf-1449-2ddb-d940-644e420351cf.png)

`ECR`に PUSH されて、 CDK のデプロイが完了しました。

![スクリーンショット 2020-12-04 0.14.39.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/855a1cae-8b42-3598-d41f-0215ecf657cd.png)


## 2.6 確認

実際に Lambda の画面にて確認してみましょう。
入力した`event`の値がそのまま返ってきていたり、`pandas.__version__`が表示されており、
`docker/app.py`で設定した通りに動作していることがわかりますね。

![スクリーンショット 2020-12-04 0.03.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/ad5e6a39-2353-df01-1263-901a34fb27bc.png)

また、実行してみたところ、初回こそ起動にちょっと時間（３秒程度）がかかりましたが、
２回目以降は即時に結果が返ってきました。

`ECR`の画面を確認すると`repository_name`にて記述した名前で登録されていました。
![スクリーンショット 2020-12-04 0.17.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/62ea0a81-6d7c-fa18-f541-2c5219c6ad15.png)


## 2.7 リソースの削除

最後に、作成したリソースは以下のコマンドで削除するようにしましょう。

```
$ cdk destroy
```

ただし、`ECR`のリポジトリまでは削除されないので、不要な場合は手動で削除するようにしてください。

# 3. おわりに

Lambda は Lambda なので実行時間などの観点から利用どころを見極める必要はあります。
ただ、Lambda でもコンテナを利用できるようになったことで、他のコンテナのサービスに柔軟に移行できるようになるのは非常に嬉しいですね。

どちらかというと、今までは Lambda 用に独自のデプロイ環境などを用意する必要がありましたが、
それがなくなったことでより多くの人が触りやすくなったと思います。

これから、`reInvent`では４つの KeyNote もありますし、2020年もまだまだ楽しめますね。
