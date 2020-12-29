# 1. はじめに

みなさん、AWS してますか？
値段が安く・触りやすく・記事も豊富にあるので、僕はかなり好きです。

ただそうは言っても、個人で触りたいとなると料金の面で利用できるものとそうでない物が出てきたり、
一旦作成したサービスを忘れることなく全て削除したりするのは面倒な気がします。
また、 Web のコンソールや CLI で作成した物は属人性が高くなり、手順やノウハウが残らないためよくないです。

そこで、便利なのが CDK です。

# 2. CDKとは

AWS が公式に提供する`Infrastructure as Code(=IaC)`のライブラリです（[GitHub](https://github.com/aws/aws-cdk)）。
CloudFormation をラップしており、`Typescript`や`Python`などを用いて実装することで、サービスの作成が可能です。

`IaC`としては、前述の CloudFormation や Terraform などもあります。
ただこれらは`json`で記述する必要があり（VSCode だとプラグインで入力補助もあります）、
作成するサービスの量が多くなると見通しが悪くなる可能性があります。


## 2.1 CDKのメリット

* AWS が公式に提供しているので利用できなくなるなどの不安はない（と思う）
* ドキュメントをみながら必要なパラメータを付与していくことになるので、必要な設定などがわかりやすい
* スタックの名前を変えれば同じ内容のサービスを複数個展開できるため、開発と本番の環境分離なども容易に行える
    * ただし、同一の名前のサービスはできないので、環境ごとに名前を変える必要な場合がある（基本的には）
* リソースの作成・削除が容易（`IaC`のメリット）
* コードでサービスの管理ができるので属人性を排除できる（`IaC`のメリット）

## 2.2 CDKのデメリット

* リリースされたばかりのサービス（機能）は対応できないことがある
    * CloudFormation が対応されたのち、 CDK の対応がされるため
    * （新サービスの LambdaContainer は４時間くらいで更新されていたので、必ずしもそうとは言い切れなくなりましたね笑）
* リリースの速度がかなり早く、過去に作成したコードが利用できなる可能性がある
    * 2020年12月07日時点で、6日に1度のペースでリリース
    * （とはいえ、破壊的な変更はそこまでない印象）
* ローカルからのコマンドでの削除が失敗することがある
    * IAM に CDK で管理していない Policy を付与などした場合

# 3. さわってみる

メリット・デメリットは実際に触ってみて感じるのが一番なので、利用方法を書いていきます。

## 3.1 環境構築

Mac の場合は`brew`にて CDK をインストールできます。

```shell
$ brew install aws-cdk
```

その後、各プロジェクトに必要な CDK の Python ライブラリを入れていく形になります。

```shell
$ pip install aws-cdk.aws-lambda
```

必要になったら、都度パッケージを入れていくことになるのですが、
`pypi aws_cdk athena`などと必要なサービス名を含めて調べるとどのパッケージを入れればいいかわかります。

## 3.1 公式サンプルプロジェクト



ゼロから、このライブラリをドキュメントから利用方法を探っていくのはかなり大変だと思うので、その際に有用なのがサンプルプロジェクトです。

AWS の公式リポジトリの[aws-cdk-example](https://github.com/aws-samples/aws-cdk-examples)が一番信頼できます。
ただ、Python のプロジェクトとなると意外と少なかったり、更新日時が古く動かないサンプルなどもあります。

## 3.2 作成したサンプルプロジェクト

そのため、私も同じようなフォーマットで、
Python のみのサンプルプロジェクト一覧[aws-cdk-small-examples](https://github.com/gsy0911/aws-cdk-small-examples)を作成してみました。
記事作成時点は CDK のバージョンが`1.76.0`なのですが、その時点で全てのサンプルプログラムが動くようになっています。
作成してみたサンプルプロジェクトの数は１２個です。
（本家公式は２５個なので、半分程度はあります。ただ、かぶっているサンプルは２個程度のみです）

対応しているサービスなどは以下の通りです。

* Lambda
* VPC
* AWS Batch
* Step Functions
* DynamoDB
* API Gateway
* CloudFront
* S3
* Glue

作成したリポジトリにて、公開しているサービスとその関連を表した図です。
（※注意：これらのサービスが一度にデプロイされるわけではありません）

![aws-cdk-small-examples-overall.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/498e3f55-07b1-82fe-88ee-a3537d9ff598.png)

次の節から公開している、構成図の一部を紹介します。

### 3.2.1 API Gateway + Lambda + DynamoDB

[この例](https://github.com/gsy0911/aws-cdk-small-examples/tree/master/python/apigw-dynamodb-lambda)では、３つの Lambda を作成し、DynamoDB にアクセスする app.py を公開しています。

![aws-cdk-small-examples-apigw_dynamodb_lambda.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/82bfb098-8c72-0e43-5109-4477cd3980f6.png)

### 3.2.2 Glue + StepFunctions

[この例](https://github.com/gsy0911/aws-cdk-small-examples/tree/master/python/glue-stepfunctions)では、Glue の`Job`を定義して、それを StepFunctions から実行する app.py を公開しています。
意外と Glue の例は少ないので、もし利用するような方がいれば参考になるかなと思います。

![aws-cdk-small-examples-glue_stepfunctions.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/05e92698-3e98-581d-7ac0-a664cf67eb01.png)

### 3.2.3 Lambda Container

[ここ](https://github.com/gsy0911/aws-cdk-small-examples/tree/master/python/lambda-container)では、`reInvent2020`で発表された新サービスの１つである、Lambda Container の例を公開しています。

細かい使い方などは別の[Qiitaの記事](https://qiita.com/gsy0911/items/943533045bec5d7de2b1)でも紹介しておりますで、合わせてどうぞ。

### 3.2.4 AWS Batch + StepFunctions

[この例](https://github.com/gsy0911/aws-cdk-small-examples/tree/master/python/batch-stepfunctions)では、AWS Batch を StepFunctions と CloudWatch Event を利用して定期実行する app.py を公開しています。
VPC なども一から作成しており、結構長いコードとなっております。
細かい説明などは、[Qiitaの記事](https://qiita.com/gsy0911/items/64d0f79601cb906cd235)にて、紹介しております。


![aws-cdk-small-examples-batch-stepfunction.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/9c861486-8cc3-b456-a598-74df7d072978.png)

### 3.2.5 NESTED AWS Batch + StepFunctions

CloudFormation が`Nest`に対応したので、3.2.4 の構成を`Nest`で書き換えた[例](https://github.com/gsy0911/aws-cdk-small-examples/tree/master/python/batch-stepfunctions-nested)を作成しております。
`Nest`を利用することのメリットは、作成した`Stack`の責任を明確にできることです。

例では、以下のように分離させています。
* VPC のスタック：ベース
* Bathc のスタック：処理などを実際に行う部分
* StepFunctions + CloudWatch Event のスタック：定期的に処理を走らせる部分


![aws-cdk-small-examples-batch-stepfunction-nested.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/9a69d2a8-6512-6d58-dd5c-4592342ba137.png)


## 3.3 おすすめの記事

* [API Reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html)
    * こちら本家の API reference です。Python 専用の docs もありますが、こちらの方が参考になります。

* [AWS CDKで快適なクラウド開発](https://qiita.com/ufoo68/items/d06756b6e7bb97359074)
* [AWS CDK(Cloud Development Kit)の参考リンク集](https://qiita.com/si1242/items/dea087a09a8a18a7d1eb)



# 4. おわりに

自分の中で、CDK はかなり好きなので、いろんな人に CDK を使っていただきたいなと思います。
`IaC`を進めるという意味でもかなり良いです。

また、これからも随時[GitHub](https://github.com/gsy0911/aws-cdk-small-examples)のサンプルは増やしていく予定なので、ぜひチェックなどをお願いいたします。
