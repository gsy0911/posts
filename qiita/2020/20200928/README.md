# 1. はじめに

今回は、AWS Batch の環境を CDK を利用して実装していきます。
よく TypeScript での実装例はあるのですが、Python はあまりなかったので記事にしました。


## 1.1 実行環境

実行環境は以下の通りです。
特にインストールや aws-cli と aws-cdk の初期設定については触れません。
ただ、注意点として aws-cdk は非常にバージョンの更新頻度が高く、現在書かれている内容でも動かない可能性があります。

* Mac OS: Catarila (10.15.6)
* Python (brew): 3.8
* aws-cli (brew): 2.0.50
* aws-cdk (brew): 1.63.0 (build 7a68125)
* docker: 19.03.12

## 1.2 料金

気になるのは、料金ですよね。
以下の条件で動かしたところ、課金されたのは EC2 の料金のみで`0.01 [$/日]`程度でした。
（Batch だと、毎回ジョブにキューが追加されてからインスタンスの作成が行われ、ジョブが完了すると削除されるためです）

* 処理時間：10~15 [min/回]
* 起動したインスタンスタイプ： c4.large（スポット料金）


## 1.3 手順

Batch の実行環境を整えるのに以下の手順で実装します。

1. Python スクリプトの作成
1. Dockerfile の作成
1. ECR に登録
1. CDK にて app を記述

## 1.4 準備

フォルダ構成は以下の通りです。
ファイル名の右側に付いているのは、上記の手順の番号と対応しています。

```shell
batch_example
└── src
    ├── docker
    │   ├── __init__.py (1)
    │   ├── Dockerfile (2)
    │   ├── requirements.txt (2)
    │   └── Makefile (3)
    └── batch_environment
        ├── app.py (4)
        ├── cdk.json
        └── README.md
```


# 2. 実装

それでは、上記の手順にしたがって実装を進めていきます。

## 2.1 Python スクリプトの作成

Docker 内にて実行するスクリプトの例を以下に示します。
`click`はコマンドライン引数の受け渡しをするために、
`watchtower`は CloudWatch Logs へのログの書き込みをするために利用しています。

```python:__init__.py
# timeのparse用
from datetime import datetime
from logging import getLogger, INFO
# インストールライブラリ
from boto3.session import Session
import click
import watchtower


# envvarで指定すると環境変数から値を取得する
@click.command()
@click.option("--time")
@click.option("--s3_bucket", envvar='S3_BUCKET')
def main(time: str, s3_bucket: str):
    if time:
        # CloudWatch Eventから実行することを想定し、時刻のparseをする
        d = datetime.strptime(time, "%Y-%m-%dT%H:%M:%SZ")
        # 実行日付を取得
        execute_date = d.strftime("%Y-%m-%d")

    # loggerの設定
    # loggerの名前がログストリームの名前になる
    logger_name = f"{datetime.now().strftime('%Y/%m/%d')}"
    logger = getLogger(logger_name)
    logger.setLevel(INFO)
    # CloudWatch Logsのロググループの名前をここで指定
    # Sessionを渡してIAM Role経由でログを送信
    handler = watchtower.CloudWatchLogHandler(log_group="/aws/some_project", boto3_session=Session())
    logger.addHandler(handler)

    # 実行予定の処理
    # ここでは、CloudWatch Logsに実行日時を書き込むのみ
    logger.info(f"{execute_date=}")
    
if __name__ == "__main__":
    """
    python __init__.py 
        --time 2020-09-11T12:30:00Z
        --s3_bucket your-bucket-here
    """
    main()
```

## 2.2 Dockerfileの作成

次に上記の Python スクリプトを実行する Dockerfile を作成します。
[ここ](https://future-architect.github.io/articles/20200513/)を参考にマルチステージでビルドしました。

```Dockerfile:Dockerfile
# ここはビルド用のコンテナ
FROM python:3.8-buster as builder

WORKDIR /opt/app

COPY requirements.txt /opt/app
RUN pip3 install -r requirements.txt

# ここからは実行用コンテナの準備
FROM python:3.8-slim-buster as runner

COPY --from=builder /usr/local/lib/python3.8/site-packages /usr/local/lib/python3.8/site-packages
COPY src /opt/app/src

WORKDIR /opt/app/src
CMD ["python3", "__init__.py"]
```

同時に`requirements.txt`にも利用するライブラリを入れておきます。

```txt:requirements.txt
click
watchtower
```


## 2.3 ECRへの登録

Dockerfile の作成が終わったら、ECR に登録します。
まずは、コンソールから ECR に「リポジトリ作成」ボタンを押してリポジトリを作成します。

![スクリーンショット 2020-09-27 22.47.35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/c189667c-4b0e-f1b5-3b38-69fb21f6dd4d.png)

リポジトリの名前は適当に設定します。

![スクリーンショット 2020-09-27 22.49.17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/ce17c5bc-b2fb-595d-76e5-f02b21888e4d.png)

作成したリポジトリを選択して、「プッシュコマンドの表示」ボタンを押します。

![スクリーンショット 2020-09-27 22.50.23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/df6dcbeb-92d7-1186-7bea-97a04d0d5dc0.png)

プッシュに必要なコマンドが表示されるので、**何も考えずにコピーして実行していきます**。
ここで、失敗する方は AWS CLI の設定がうまくできていないので、AWS CLI の設定を見直してください。

![スクリーンショット 2020-09-27 22.51.53.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/d4dc6fa7-8d78-c2bf-e28f-6203bd461692.png)


毎回、コマンドを打つのが大変なので、上記のコマンドをコピーした`Makefile`を作成します。
（１のコマンドの`--username AWS`は定数だと思われます）

```txt:Makefile
.PHONY: help
help:
	@echo " == push docker image to ECR == "
	@echo "type 'make build tag push' to push docker image to ECR"
	@echo ""

.PHONY: login
login:
	(1のコマンド)aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin {ACCOUNT_NUMBER}.dkr.ecr.ap-northeast-1.amazonaws.com

.PHONY: build
build:
	(2のコマンド)docker build -t {REPOSITORY_NAME} .

.PHONY: tag
tag:
	(3のコマンド)docker tag {REPOSITORY_NAME}:latest {ACCOUNT_NUMBER}.dkr.ecr.ap-northeast-1.amazonaws.com/{REPOSITORY_NAME}:latest

.PHONY: push
push:
	(4のコマンド)docker push {ACCOUNT_NUMBER}.dkr.ecr.ap-northeast-1.amazonaws.com/{REPOSITORY_NAME}:latest
```

この`Makefile`を利用することで、以下のようにコマンドを実行できます。
加えて、上記の`Makefile`には特に外部にもれても危ない情報はないので、ソースコードも共有できます。

```shell
# ECRにログイン
$ make login

# ECRに最新の状態のimageをpush
$ make build tag push
```

## 2.4 CDKの実装

CDK の実装内容は TypeScript で書かれた[この記事](https://qiita.com/ishizakit/items/64ea4b5acd9637149360)を参考に行いました。
なお、事前に app.py を実装するディレクトリにて`$ cdk init`の実行をした方が良いです。

### 2.4.1 実装に必要なパッケージのインストール

ひとつひとつのパッケージ名が長いですね。
加えて、インストールにかかる時間も結構長いです。

```shell
$ pip install aws-cdk-core aws-cdk-aws-stepfunctions aws-cdk-aws-stepfunctions-tasks aws-cdk-aws-events-targets aws-cdk.aws-ec2 aws-cdk.aws-batch aws-cdk.aws-ecr
```

### 2.4.2 app.pyの実装

まずは、今回構築する環境のクラスを作成します。
BatchEnvironment クラスの引数として、`stack_name`と`stack_env`を設定しています。
これは、この環境の名前と、実行環境（検証／開発／本番）などと対応しています。
（なお、本当に実行環境を分ける場合は、ECR のリポジトリも変更した方が良いです）

```python:app.py
from aws_cdk import (
    core,
    aws_ec2,
    aws_batch,
    aws_ecr,
    aws_ecs,
    aws_iam,
    aws_stepfunctions as aws_sfn,
    aws_stepfunctions_tasks as aws_sfn_tasks,
    aws_events,
    aws_events_targets,
)


class BatchEnvironment(core.Stack):
    """
    Batchの環境とそれを実行するStepFunctions + CloudWatch Event環境を作成

    """
    # 上で作成したECRのリポジトリ名
    # Batchで実行する際に、このリポジトリからイメージをpullする
    ECR_REPOSITORY_ARN = "arn:aws:ecr:ap-northeast-1:{ACCOUNT_NUMBER}:repository/{YOUR_REPOSITORY_NAME}"

    def __init__(self, app: core.App, stack_name: str, stack_env: str):
        super().__init__(scope=app, id=f"{stack_name}-{stack_env}")
        # 以下の実装はここの下に連なるイメージです。

```

### 2.4.3 app.pyの実装（VPC環境の作成）

```python:app.py
        # def __init__(...):の中

        # CIDRは好きな範囲を
        cidr = "192.168.0.0/24"

        # === #
        # vpc #
        # === #
        # VPCはパブリックサブネットしか利用しない場合は、無料で利用可能できる（はずです）
        vpc = aws_ec2.Vpc(
            self,
            id=f"{stack_name}-{stack_env}-vpc",
            cidr=cidr,
            subnet_configuration=[
                # Public Subnetのnetmaskを定義
                aws_ec2.SubnetConfiguration(
                    cidr_mask=28,
                    name=f"{stack_name}-{stack_env}-public",
                    subnet_type=aws_ec2.SubnetType.PUBLIC,
                )
            ],
        )

        security_group = aws_ec2.SecurityGroup(
            self,
            id=f'security-group-for-{stack_name}-{stack_env}',
            vpc=vpc,
            security_group_name=f'security-group-for-{stack_name}-{stack_env}',
            allow_all_outbound=True
        )

        batch_role = aws_iam.Role(
            scope=self,
            id=f"batch_role_for_{stack_name}-{stack_env}",
            role_name=f"batch_role_for_{stack_name}-{stack_env}",
            assumed_by=aws_iam.ServicePrincipal("batch.amazonaws.com")
        )

        batch_role.add_managed_policy(
            aws_iam.ManagedPolicy.from_managed_policy_arn(
                scope=self,
                id=f"AWSBatchServiceRole-{stack_env}",
                managed_policy_arn="arn:aws:iam::aws:policy/service-role/AWSBatchServiceRole"
            )
        )

        batch_role.add_to_policy(
            aws_iam.PolicyStatement(
                effect=aws_iam.Effect.ALLOW,
                resources=[
                    "arn:aws:logs:*:*:*"
                ],
                actions=[
                    "logs:CreateLogGroup",
                    "logs:CreateLogStream",
                    "logs:PutLogEvents",
                    "logs:DescribeLogStreams"
                ]
            )
        )

        # EC2に付与するRole
        instance_role = aws_iam.Role(
            scope=self,
            id=f"instance_role_for_{stack_name}-{stack_env}",
            role_name=f"instance_role_for_{stack_name}-{stack_env}",
            assumed_by=aws_iam.ServicePrincipal("ec2.amazonaws.com")
        )

        instance_role.add_managed_policy(
            aws_iam.ManagedPolicy.from_managed_policy_arn(
                scope=self,
                id=f"AmazonEC2ContainerServiceforEC2Role-{stack_env}",
                managed_policy_arn="arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceforEC2Role"
            )
        )

        # S3にアクセスするpolicyの追加
        instance_role.add_to_policy(
            aws_iam.PolicyStatement(
                effect=aws_iam.Effect.ALLOW,
                resources=["*"],
                actions=["s3:*"]
            )
        )

        # CloudWatch Logsにアクセスするpolicyの追加
        instance_role.add_to_policy(
            aws_iam.PolicyStatement(
                effect=aws_iam.Effect.ALLOW,
                resources=[
                    "arn:aws:logs:*:*:*"
                ],
                actions=[
                    "logs:CreateLogGroup",
                    "logs:CreateLogStream",
                    "logs:PutLogEvents",
                    "logs:DescribeLogStreams"
                ]
            )
        )

        # EC2にロールを付与
        instance_profile = aws_iam.CfnInstanceProfile(
            scope=self,
            id=f"instance_profile_for_{stack_name}-{stack_env}",
            instance_profile_name=f"instance_profile_for_{stack_name}-{stack_env}",
            roles=[instance_role.role_name]
        )
```

###　2.4.4 app.py の実装（Batch の実行環境およびジョブ定義・ジョブキューの作成）

```python:app.py
        # VPCの続き...

        # ===== #
        # batch #
        # ===== #
        batch_compute_resources = aws_batch.ComputeResources(
            vpc=vpc,
            maxv_cpus=4,
            minv_cpus=0,
            security_groups=[security_group],
            instance_role=instance_profile.attr_arn,
            type=aws_batch.ComputeResourceType.SPOT
        )

        batch_compute_environment = aws_batch.ComputeEnvironment(
            scope=self,
            id=f"ProjectEnvironment-{stack_env}",
            compute_environment_name=f"ProjectEnvironmentBatch-{stack_env}",
            compute_resources=batch_compute_resources,
            service_role=batch_role
        )

        job_role = aws_iam.Role(
            scope=self,
            id=f"job_role_{stack_name}-{stack_env}",
            role_name=f"job_role_{stack_name}-{stack_env}",
            assumed_by=aws_iam.ServicePrincipal("ecs-tasks.amazonaws.com")
        )

        job_role.add_managed_policy(
            aws_iam.ManagedPolicy.from_managed_policy_arn(
                scope=self,
                id=f"AmazonECSTaskExecutionRolePolicy_{stack_name}-{stack_env}",
                managed_policy_arn="arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"

            )
        )

        job_role.add_managed_policy(
            aws_iam.ManagedPolicy.from_managed_policy_arn(
                scope=self,
                id=f"AmazonS3FullAccess_{stack_name}-{stack_env}",
                managed_policy_arn="arn:aws:iam::aws:policy/AmazonS3FullAccess"

            )
        )

        job_role.add_managed_policy(
            aws_iam.ManagedPolicy.from_managed_policy_arn(
                scope=self,
                id=f"CloudWatchLogsFullAccess_{stack_name}-{stack_env}",
                managed_policy_arn="arn:aws:iam::aws:policy/CloudWatchLogsFullAccess"
            )
        )

        batch_job_queue = aws_batch.JobQueue(
            scope=self,
            id=f"job_queue_for_{stack_name}-{stack_env}",
            job_queue_name=f"job_queue_for_{stack_name}-{stack_env}",
            compute_environments=[
                aws_batch.JobQueueComputeEnvironment(
                    compute_environment=batch_compute_environment,
                    order=1
                )
            ],
            priority=1
        )

        # ECRリポジトリの取得
        ecr_repository = aws_ecr.Repository.from_repository_arn(
            scope=self,
            id=f"image_for_{stack_name}-{stack_env}",
            repository_arn=self.ECR_REPOSITORY_ARN
        )

        # ECRからイメージの取得
        container_image = aws_ecs.ContainerImage.from_ecr_repository(
            repository=ecr_repository
        )

        # ジョブ定義
        # ここで、Python scriptで利用する`S3_BUCKET`を環境変数として渡す
        batch_job_definition = aws_batch.JobDefinition(
            scope=self,
            id=f"job_definition_for_{stack_env}",
            job_definition_name=f"job_definition_for_{stack_env}",
            container=aws_batch.JobDefinitionContainer(
                image=container_image,
                environment={
                    "S3_BUCKET": f"{YOUR_S3_BUCKET}"
                },
                job_role=job_role,
                vcpus=1,
                memory_limit_mib=1024
            )
        )


```

### 2.4.5 app.pyの実装（StepFunctions + CloudWatch Eventsの作成）

ここからは、必ずしも Batch の環境構築に必要ではありませんが、
定期実行をするために StepFunctions と CloudWatch Event を利用して行います。

CloudWatch Event からも直接 Batch を呼べますが、
他サービスとの連携のしやすさやパラメータの受け渡しなどを考えて間に StepFunctions を挟んでいます。

StepFunctions のステップとして登録する際に、
Docker の CMD コマンドを（＝Batch のジョブ定義に設定した状態）上書きします。
そして、CloudWatch Event からの引数`time`を受け取り、Python スクリプトへ渡しています。

```python:app.py
        # Batchの続き...

        # ============= #
        # StepFunctions #
        # ============= #

        command_overrides = [
            "python", "__init__.py",
            "--time", "Ref::time"
        ]

        batch_task = aws_sfn_tasks.BatchSubmitJob(
            scope=self,
            id=f"batch_job_{stack_env}",
            job_definition=batch_job_definition,
            job_name=f"batch_job_{stack_env}_today",
            job_queue=batch_job_queue,
            container_overrides=aws_sfn_tasks.BatchContainerOverrides(
                command=command_overrides
            ),
            payload=aws_sfn.TaskInput.from_object(
                {
                    "time.$": "$.time"
                }
            )
        )

        # 今回は1ステップしかないので、単純ですが複数のステップをつなげたい場合は
        # batch_task.next(aws_sfn_tasks.JOB).next(aws_sfn_tasks.JOB)
        # のようにチェインメソッドで渡せます。
        definition = batch_task

        sfn_daily_process = aws_sfn.StateMachine(
            scope=self,
            id=f"YourProjectSFn-{stack_env}",
            definition=definition
        )

        # ================ #
        # CloudWatch Event #
        # ================ #

        # Run every day at 21:30 JST
        # See https://docs.aws.amazon.com/lambda/latest/dg/tutorial-scheduled-events-schedule-expressions.html
        events_daily_process = aws_events.Rule(
            scope=self,
            id=f"DailySFnProcess-{stack_env}",
            schedule=aws_events.Schedule.cron(
                minute=31,
                hour=12,
                month='*',
                day="*",
                year='*'),
        )
        events_daily_process.add_target(aws_events_targets.SfnStateMachine(sfn_daily_process))

        # ここまで def __init__(...):
```


### 2.4.6 app.pyの実装（main関数の実装）

最後に、CDK を実行する処理を書いたら、完了です。

```python:app.py
# ここに def __init__(...):


def main():
    app = core.App()
    BatchEnvironment(app, "your-project", "feature")
    BatchEnvironment(app, "your-project", "dev")
    BatchEnvironment(app, "your-project", "prod")
    app.synth()


if __name__ == "__main__":
    main()

```

## 2.5 デプロイ

上記のスクリプトが完成後に、以下のコマンドで正しく CDK の設定ができているか確認の上、デプロイしましょう。
Batch の環境を0から作成する場合でも１０分程度で完了します。

```shell
# 定義の確認
$ cdk synth

Successfully synthesized to {path_your_project}/cdk.out
Supply a stack id (your-project-dev, your-project-feature, your-project-prod) to display its template.

# デプロイできる環境の確認
$ cdk ls

your-project-dev
your-project-feature
your-project-prod

$ cdk deploy your-project-feature

...deploying...

```

### 2.5.1 環境が正しく作られたか確認する

デプロイが完了したら、コンソールから作成した StepFunctions を選択し、「実行の開始」ボタンを押します。

![スクリーンショット 2020-09-27 23.38.58.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/28bf4603-2cb2-455c-f310-2d2696822930.png)


`time`の引数だけ入れます。

```
{
    "time": "2020-09-27T12:31:00Z"
}
```

![スクリーンショット 2020-09-27 23.45.12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/260295/fbec1a43-215f-e8d6-11de-40e36db6bb35.png)

正しく動いたら完了です。
また、CloudWatch Logs へも想定した通りに動いているか確認しましょう。

# 3. おわりに

CDK は、環境の構築と削除がコマンドですぐにできるのでめっちゃ好きです。

コンソールから作成するよりも、プログラムのパラメータで求められているものがわかるので、
どんなパラメータが必須かがわかりやすくて良いなと思いました。

また、こちらの [GitHube リポジトリ](https://github.com/gsy0911/aws-cdk-small-examples)にいろいろな CDK プロジェクトについてまとめてあります。
こちらも是非、ご覧ください。
